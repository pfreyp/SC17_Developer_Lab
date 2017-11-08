
# Experiencing F1 Acceleration

## Overview

In this module you will experience the acceleration potential of AWS F1 instances by using ```ffmpeg``` to encode 20 seconds of raw YUV 1920x1080 video, first using the libx265 codec and then the [NGCodec](https://ngcodec.com/products-cloud-transcoding/) HEVC encoder optimized for F1 FPGAs.

```ffmpeg``` is a very popular framework providing very fast video and audio converters. The ```ffmpeg``` code is open-source and allows for the addition of custom plugins. For this lab, a custom plugin has been created to transparently use the NGCodec HEVC encoder running on AWS F1. Users can switch between the libx265 software codec and the F1-accelerated implementation by simply changing a parameter on the ```ffmpeg``` command line. The plugin uses OpenCL API calls to write video frames to the FPGA, execute the encoder and read back the compressed video. 

## Running the lab

* Open a new terminal by right-clicking anywhere in the Desktop area and selecting **Open Terminal**.
* Navigate to the FFmpeg lab directory.
```
cd ~/SC17_Developer_Lab/ffmpeg
```

* Source the FFmpeg and SDAccel runtime environments.
```
sudo sh
source ./ffsetup.sh
source /opt/Xilinx/SDx/2017.1.rte/setup.sh
```

* Encode using the libx265 software codec.
```
./ffmpeg -f rawvideo -pix_fmt yuv420p -s:v 1920x1080 -i /home/centos/vectors/crowd8_420_1920x1080_50.yuv -an -frames 1000 -c:v libx265 -preset medium -g 30 -q 40 -f hevc -y ./crowd8_420_1920x1080_50_libx265_out0_qp40.hevc
```

The encoder will finish with a message similar to this one: \
*frame=500 fps=9.0 q=-0.0 Lsize=19933kB time=00:00:19.92 bitrate=8197.4kbits/s speed=0.358x* 

The libx265 encoder processed the entire video in about **55.5 seconds** (500 frames / 9 fps).


* Preload the HEVC encoder AFI in the FPGA attached to the F1 instance. 
```
fpga-load-local-image -S 0 -I agfi-0015437e933b3e725
```

* Encode using the NGCodec HEVC encoder running on the F1 FPGA.
```
./ffmpeg -f rawvideo -pix_fmt yuv420p -s:v 1920x1080 -i /home/centos/vectors/crowd8_420_1920x1080_50.yuv -an -frames 1000 -c:v xlnx_hevc_enc -psnr -g 30 -global_quality 40 -f hevc -y ./crowd8_420_1920x1080_50_NGcodec_out0_g30_gq40.hevc 
```

The encoder will finish with a message similar to this one: \
*frame=500 fps= 52 q=-0.0 LPSNR=Y:inf U:inf V:inf \*:inf size=17580kB time=00:00:20.00 bitrate=7200.9kbits/s speed=2.08x* 

The HEVC encoder running on F1 processed the same video in about **9.6 seconds** (500 frame / 52 fps), a **5.7x** performance boost over the libx265 implementation.

* Compare the file sizes of the two encoded videos
```
ls -la --block-size=K *hevc
```
Notice that the HEVC encoder running on F1 produced a denser ouput: **17.5Mb** vs **19.9Mb**.

* Open and watch the videos using ```smplayer```. Note: rendering over RDP will not be ideal, but you will nonetheless be able to compare the two encoded videos.
```
# Output from the libx265 encoder
smplayer ./crowd8_420_1920x1080_50_libx265_out0_qp40.hevc

# Output from the NGCodec encoder
smplayer ./crowd8_420_1920x1080_50_NGcodec_out0_g30_gq40.hevc 
```
Notice that the picture quality is similar for both encoders. The HEVC encoder running on F1 is not only much faster, but it also provides better compression without sacrificing quality.

Finally, a profile summary was generated when running the HEVC encoder on F1. It is a report which helps profile the performance of the application running on F1. The summary is available in both html and csv formats.

* Open the SDAccel HTML profile summary in a web browser.
```
firefox --new-tab sdaccel_profile_summary.html &
```
* At the top of the report, you will find a summary of all the **OpenCL API Calls** made by the custom ffmpeg plugin to the FPGA accelerator. On the 3rd row, notice how **clEnqueueNDRangeKernel** is executed 500 times, once per video frame. 
* Scroll down to the next table called **Kernel Execution**. Notice how the total execution time for the NGCodec kernel is about 8.9 seconds, which closely matches the 9.6 second of total ffmpeg processing time. Note also how the average processing time about 17.9 milliseconds per frame.
* The SDAccel reports will be discussed in more details in the next lab.
* Close Firefox and exit the shell to finish this lab.

## Conclusion

In this lab you have accomplished the following:
* Measured a 5.7x performance increase by using the NGCodec HEVC encoder running on F1. 
   - Multiple instances of the NGCodec encoder could be loaded in the FPGA, allowing parallel processing of multiple video streams and easily delivering more than a 10x increase in performance/$ over a CPU-based solution. 
* Learned that it is possible to use F1 to accelerate popular frameworks such as ```ffmpeg```. 
   - This is a very powerful proposition at it allows end-users to keep working with their preferred tools and APIs while transparently benefiting from acceleration.

In addition to video transcoding, F1 instances are very well suited to accelerate compute intensive workloads such as: genomics, financial analytics, big data analytics, security or machine learning.

Now that you have experienced the performance potential of AWS F1 instances, the next lab will introduce you to the SDAccel IDE and how to develop, profile and optimize an F1 application.

---------------------------------------
[**Start Module 3: Developing and optimizing F1 applications with SDAccel**](IDCT_Lab.md)
