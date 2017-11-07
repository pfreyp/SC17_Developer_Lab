
# FFMPEG Lab

Basic instructions for now. Story will follow later...

* Open a new terminal
* Navigate to the FFmpeg lab directory
```
cd ~/SC17_Developer_Lab/ffmpeg
```

* Source the environment
```
sudo sh
source ./ffsetup.sh
source /opt/Xilinx/SDx/2017.1.rte/setup.sh
```

* Preload the HEVC encoder AFI
```
fpga-load-local-image -S 0 -I agfi-0015437e933b3e725
```
* Run the SW implementation using libx265
```
./ffmpeg -f rawvideo -pix_fmt yuv420p -s:v 1920x1080 -i /home/centos/vectors/crowd8_420_1920x1080_50.yuv -an -frames 1000 -c:v libx265 -preset medium -g 30 -q 40 -f hevc -y ./crowd8_420_1920x1080_50_libx265_out0_qp40.hevc
```

The encoder will finish with message similar to this one: \
frame=  500 fps=9.0 q=-0.0 Lsize=   19933kB time=00:00:19.92 bitrate=8197.4kbits/s speed=0.358x 

We can see that the software encoder processed the 500 frames at a performance of 9 frames per second (fps), taking 55.5 sec to encode the entire video.

* Now run the F1-optimized implementation using the NGCodec HEVC encoder FPGA Image
```
./ffmpeg -f rawvideo -pix_fmt yuv420p -s:v 1920x1080 -i /home/centos/vectors/crowd8_420_1920x1080_50.yuv -an -frames 1000 -c:v xlnx_hevc_enc -psnr -g 30 -global_quality 40 -f hevc -y ./crowd8_420_1920x1080_50_NGcodec_out0_g30_gq40.hevc 
```

The encoder will finish with message similar to this one: \
frame=  500 fps= 52 q=-0.0 LPSNR=Y:inf U:inf V:inf *:inf size=   17580kB time=00:00:20.00 bitrate=7200.9kbits/s speed=2.08x 

The F1-optimized encoder processed the video in about 9.6 seconds at a performance of 52 fps, a 5.7x performance boost over the software implementation.

* Look at the SDAccel profiling report
```
firefox --new-tab sdaccel_profile_summary.html
```
- - -

Legacy instructions here



## Contents
1. [Introduction](#Introduction)
1. [Code Hierarchy](#CodeHierarchy)
1. [FFMPEG Framework Example](#FrameworkExample)
1. [FFMPEG HEVC Acceleration](#HEVCAcceleration)

<a name="Introduction"></a>
## Introduction

This lab is intended to illustrate how an FPGA accelerator can be
deployed on AWS-F1. The example is based on the FFMPEG framework which
is designed to allow easy integration of various video formats and
image manipulation algorithms.

The lab starts with a brief introduction to the code hierarchy. This
is followed by a copy kernel example. We will first examine a software
version before integrating an existing AFI providing the same copy
functionality. This exercise will focus on the OpenCL integration
layer located between the ffmpeg interface and the kernel
execution. Note, this example is not expected to show any acceleration
as basically no computation is performed.

In the next step, we perform a more realistic comparison between an
accelerated implementation and a SW version of an HEVC codec. This is
based on the same FFMPEG framework in which both versions are
integrated.

<a name="CodeHierarchy"></a>
## Code Hierarchy

The FFMPEG source code as well as the source code dependencies are
already installed on the tutorial machines. Similarly, test vectors
to exercise the different video encoders are installed as well. The
respective directories are:

``` shell
~/vectors
~/ffmpeg
```

To keep a clean build hierarchy, the FFMPEG source tree is located in

``` shell
~/ffmpeg/sources/ffmpeg
```
 
This code was cloned directly from github

``` shell
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
```

This code was then modified by applying overlays and adding software
packages to support a simple copy kernel implementation, a software
version of the HEVC algorithm, and an FPGA based acceleration of the
HEVC algorithm. These integrations required only changes to the
Makefile and the codec source files in the following directory:

``` shell
~/ffmpeg/sources/ffmpeg/libavcodec/
```

In the next section, we will first look at the FFMPEG interface before
modifying it to actually call an equivalent hardware acceleration
function.

<a name="FrameworkExample"></a>
## FFMPEG Framework Example

This lab section can be split into two sections. The first part
introduces the FFMPEG framework interface. This will be performed with
the help of a simple software copy function implementation. In the
second part, this function is modified to execute a hardware copy
function implementation.

### Software Copy Function

The number of files in the libavcodec directory is &gt; 1000. Later in
the build process, you will see that there are hundreds of codecs
supported. The build process for all of them is controlled through the
Makefile in this directory. 

``` shell
cd ~/ffmpeg/sources/ffmpeg/libavcodec
more Makefile
```

In the makefile only the Object files (.o) are needed to be added to
include them in the build process. The actual registration of the new
algorithm with the executable is performed in the allcodecs.c
file. For example the software copy kernel registration is performed
by the following **REGISTER_ENCODER** macro.

``` shell
$ grep xlnx_copy_sw allcodecs.c
    REGISTER_ENCODER(XLNX_COPY_SW,      xlnx_copy_sw);
```

Through this registration process, the FFMPEG framework will be
looking for a global instance of a AVCodec structure. This can be
found in the file xlnx_copy_sw_enc.c file.

``` C
AVCodec ff_xlnx_copy_sw_encoder = {
    .name           = "xlnx_copy_sw_enc",
    .long_name      = NULL_IF_CONFIG_SMALL("Xilinx Example Encoder"),
    .type           = AVMEDIA_TYPE_VIDEO,
    .id             = AV_CODEC_ID_XLNX_COPY,
    .init           = xlnx_copy_sw_encode_init,
    .encode2        = xlnx_copy_sw_encode_frame,
    .close          = xlnx_copy_sw_encode_close,
    .capabilities   = CODEC_CAP_DR1,
    .pix_fmts       = (const enum AVPixelFormat[])
    {
        AV_PIX_FMT_YUV420P, AV_PIX_FMT_NONE
    },
};
```

This structure provides many fields well defined in the FFMPEG
documentation. For this training, the most important ones are the
registered function handles init, encode2, and close. These functions
are locally defined in this file but are in principle just forwarding
the call to the actual implementation in the **sw_copy.c** file.

``` SHELL
cd ~/ffmpeg/sources/ffmpeg/libavcodec/xlnx_copy_sw
more sw_copy.c
```

The init function (**ff_init_copy_sw_xlnx**) is called once before the
encoding process starts. In this simple case, it sets up the global
static variables required for the encoding process. Note, the AVCodec
handle can also be utilized to manage data local to the
codec. However, in this case global static variables were chosen to
enhance readability.

Next we have the actual encode function (**ff_copy_sw_xlnx**). This
function simply copies the incoming image (**pic**) to the destination
(**dst**) vector.

Finally there is the function (**ff_uninit_copy_sw_xlnx**) associated
with the close function handle. This function is called once after the
encoding process has completed. The idea of this function handle will
become clear once internal data structures are required to be cleaned
up.

To complete this simple coding example, we need to understand how do
build the complete framework and dependencies. For this purpose, a
simple build script is provided and can be executed as follows:

``` shell
cd ~/ffmpeg
./build_ffmpeg.sh
```

Once completed the ffmpeg executable can be exercised with the help of
the following command.

``` shell
sudo ./run.sh cp_sw.cmd
```

Note, **sudo** is not strictly required in this case as no FPGA
accelerator is executed. However, we include it here for
consistency. The actual call to the ffmpeg executable is done through
**cp_sw.cmd**.

``` shell
more cp_sw.cmd

./ffmpeg/sources/ffmpeg/ffmpeg_build/ffmpeg \
-f rawvideo -pix_fmt yuv420p -s:v 1920x1080 -r 30 -an \
-i ../vectors/bbb_sunflower_1080p_30fps_normal_1000frames.yuv \
-frames 1000 -global_quality -1 -b:v 4000k -g 30 -c:v xlnx_copy_sw_enc \
-f rawvideo -pix_fmt yuv420p -s:v 1920x1080 -y ./sw_outdir/bbb_sunflower_1k1c_ou
t.yuv
```

Please note, among all the command options there is the input file
defined (**-i**), the output file (**-y**), and the selection of the
encoder (**-c:v**).

Since this is a simple copy kernel, the output and the inputs should
be equivalent. This can be confirmed through the Linux diff command:

``` shell
diff sw_outdir/bbb_sunflower_1k1c_out.yuv ../vectors/bbb_sunflower_1080p_30fps_normal_1000frames.yuv
```


### Hardware Copy Function

Now that we had a brief introduction to the FFMPEG framework, we will
be extending the example to actually interact with an FPGA based
accelerator. Towards that end, we will only need to modify the
(**sw_copy.c**) file with your favorite editor.

``` SHELL
cd ~/ffmpeg/sources/ffmpeg/libavcodec/xlnx_copy_sw
<editor> sw_copy.c
```

If you look at the file, you will see five locations of the form

``` C
// ----- Topic ---------------------------------------------------------------
// ----- Topic ---------------------------------------------------------------
```

Where topic is either **Static Variables**, **OpenCL
Context**,**OpenCL Initialization, Buffer Setup, Kernel Argument
Setting**, **Execute the HW kernel**, and **Release OpenCl
references**. We will be discussing each of these sections and you
are asked to copy the code from this document into the source
file between the appropriate comment lines.

The first section is concerned with the internal state of this
encoder. Here we declare global static variables local to this file,
responsible to keep state between the different function calls of the
encoder.

``` C
// ----- Static Variables ----------------------------------------------------
//Globals - for opencl
static AVOpenCLExternalEnv *cl_env;
static cl_kernel kernel = NULL;
static cl_program program = NULL;

// OpenCL Buffers
static cl_mem INPUT_FP_Y_V;
static cl_mem INPUT_FP_U_V;
static cl_mem INPUT_FP_V_V;
static cl_mem BITSTREAM_BASE_V;
static cl_mem REF_BASE_V;
static cl_mem BITSTREAM_LEN_V;
static cl_mem DUMMY_CNT;
static cl_int status;
// ----- Static Variables ----------------------------------------------------

```

These are the usual OpenCL candidates, maintaining pointers to the
OpenCL objects such as the execution kernel and buffers.

The next section contains two static functions. The first is
responsible to read the content of a file into memory and return the
total size. The second utilizes the first function and performs all
the common OpenCL setup tasks.

``` C
// ----- OpenCL Context ------------------------------------------------------
// Simple load file to memory function
static int load_file_to_memory_copy(const char *filename, unsigned char **result)
{
   int size = 0;
   FILE *f = fopen(filename, "rb");
   if(f == NULL)
   {
       *result = NULL;
        return -1; // -1 means file opening fail
   }
   fseek(f, 0, SEEK_END);
   size = ftell(f);
   fseek(f, 0, SEEK_SET);
   *result = malloc(size);
   if(size != fread(*result, sizeof(char), size, f))
   {
      free(*result);
      return -2; // -2 means file reading fail
   }
   fclose(f);
   return size;
}

// The following function performs the OpenCL context creation, populating 
// the static structures:
//
// static AVOpenCLExternalEnv *cl_env;
// static cl_kernel kernel;
// static cl_program program; 
//
static int allocate_opencl_env() {

    unsigned char *kernelbinary = 0;
    size_t kernel_len = 0;

    cl_env = av_opencl_alloc_external_env();
    cl_platform_id platforms[16] = { 0 };
    cl_device_id devices[16];
    char platformName[256];
    char deviceName[256];
    cl_uint platformCount = 0;
    cl_int err = clGetPlatformIDs(0, 0, &platformCount);
    err = clGetPlatformIDs(16, platforms, &platformCount);
    if (err != CL_SUCCESS) {
        return -1;
    }
    cl_env->device_type = CL_DEVICE_TYPE_ACCELERATOR;

    for (cl_uint i = 0; i < platformCount; i++) {
      err = clGetPlatformInfo(platforms[i], CL_PLATFORM_NAME, 256, platformName, 0);
      if (err != CL_SUCCESS) {
            return -1;
      }
      cl_uint deviceCount = 0;
      err = clGetDeviceIDs(platforms[i], cl_env->device_type, 16, devices, &deviceCount);
      if ((err != CL_SUCCESS) || (deviceCount == 0)) {
          continue;
      }

      err = clGetDeviceInfo(devices[0], CL_DEVICE_NAME, 256, deviceName, 0);
      if (err != CL_SUCCESS) {
        return -1;
      }

      cl_context_properties contextData[3] = {CL_CONTEXT_PLATFORM, (cl_context_properties)platforms[i], 0};
      cl_context context = clCreateContextFromType(contextData, cl_env->device_type, 0, 0, &err);
      if (err != CL_SUCCESS) {
          continue;
      }
      cl_command_queue queue = clCreateCommandQueue(context, devices[0], 0, &err);
      if (err != CL_SUCCESS) {
        return -1;
      }
      cl_env->platform_id = platforms[i];
      cl_env->context = context;
      cl_env->command_queue = queue;
      cl_env->device_type = CL_DEVICE_TYPE_ACCELERATOR;
      cl_env->device_id = devices[0];
      break;
    }
    //
    // Create the program
    //
    unsigned char* xcl_path = getenv("XLNX_XCLBIN_PATH");
    if (NULL == xcl_path)
    {
      av_log(NULL, AV_LOG_INFO, " * XLNX_XCLBIN_PATH not found\n");
      return -1;
    }
    char xcl_fullpath[PATH_MAX];
    int mlen = 0;
    snprintf(xcl_fullpath, sizeof(xcl_fullpath), "%s/krnl_datamover.awsxclbin", xcl_path);
    mlen = load_file_to_memory_copy(xcl_fullpath, &kernelbinary);
    av_log(NULL, AV_LOG_INFO, " * loaded file to memory %s\n", xcl_fullpath);
    if (mlen < 0)
    {
      av_log(NULL, AV_LOG_INFO, " * loaded file to memory failed %s ---%d\n", xcl_fullpath,mlen);
      return EXIT_FAILURE;
    }

    kernel_len = mlen;
 
    program = clCreateProgramWithBinary(cl_env->context, 1, &(cl_env->device_id),
                   &kernel_len, (const unsigned char**)&kernelbinary, &status, NULL);
    if( status != CL_SUCCESS || !program )
    {
        av_log(NULL, AV_LOG_ERROR, "OpenCL unable to create fpga accelerator program\n");
        return -1;
    }

    //
    // Build the program
    //
    status = clBuildProgram(program, 1, &(cl_env->device_id), NULL, NULL, NULL);
    if( status != CL_SUCCESS )
    {
        av_log(NULL, AV_LOG_ERROR, "OpenCL unable to build program\n");
        return -1;
    }

    //
    // Create the kernel
    //
    kernel = clCreateKernel(program, "krnl_datamover", &status);
    if( status != CL_SUCCESS )
    {
        av_log(NULL, AV_LOG_ERROR, "OpenCL unable to create kernel\n");
        return -1;
    }

    return 0;
}
// ----- OpenCL Context ------------------------------------------------------

```

People familiar with OpenCL should be very familiar with these
steps. All of these interface function are documented by the OpenCL
maintainer [(Khronos group)](https://www.khronos.org). It is quite
common to create a function and reuse them between projects as only
the arguments to some operations change and the handling of the error
conditions are common.

Here we just briefly touch on each function:

1. **clGetPlatformIDs**: This function queries the system to identify
   the different OpenCL platforms.
1. **clGetPlatformInfo**: Get specific information about the OpenCL platform.
1. **clGetDeviceIDs**: Obtain the list of devices available on a platform.
1. **clGetDeviceInfo**: Extract information about OpenCL device
1. **clCreateContextFromType**: Create an OpenCL context from a device type that identifies the specific device to use.
1. **clCreateCommandQueue**: Create a command-queue on a specific device.
1. **clCreateProgramWithBinary**: Creates a program object for a context, and loads specified binary data into the program object. The actual program is obtained before this call through the load_file_to memory_copy function.
1. **clBuildProgram**: Builds a program executable from the program source or binary.
1. **clCreateKernel**: Creates a kernal object.

The key components of the setup functionality are retained in the static global variables.

The setup function is called from the initialization function
(**ff_init_copy_sw_xlnx**). Please add the following code section
between the comments:

``` C
    // ----- OpenCL Initialization, Buffer Setup, Kernel Argument Setting ----
    allocate_opencl_env(); 
    //
    // Setup copy codec core defaults
    //         
    unsigned int FRAME_WIDTH_V = avctx->width;
    unsigned int FRAME_HEIGHT_V = avctx->height;
    unsigned int FIXED_QP_V = 0;
    unsigned int BITRATE_V = 0;
    unsigned int INTRA_PERIOD_V = avctx->gop_size;
    unsigned int DUMMY_OUTRATIO = 1;
    unsigned int DUMMY_DELAY = 2000000;
    if (DUMMY_OUTRATIO == 0 ) DUMMY_OUTRATIO = 1;
    if (avctx->bit_rate > 0)
    {
      // Constant bitrate mode
      BITRATE_V = avctx->bit_rate;
    }
    else
    {
      // Fixed QP mode
      FIXED_QP_V = avctx->global_quality;
    }

    //
    // Setup OpenCL Buffers and Kernel Parameters
    //
    CREATEBUF(INPUT_FP_Y_V, CL_MEM_READ_ONLY,  CurSourceImageSize[Y] * sizeof(unsigned char));
    CREATEBUF(INPUT_FP_U_V, CL_MEM_READ_ONLY,  CurSourceImageSize[U] * sizeof(unsigned char));
    CREATEBUF(INPUT_FP_V_V, CL_MEM_READ_ONLY,  CurSourceImageSize[V] * sizeof(unsigned char));
    CREATEBUF(BITSTREAM_BASE_V, CL_MEM_WRITE_ONLY, EncodedFrameSize * sizeof(unsigned char));
    CREATEBUF(REF_BASE_V, CL_MEM_READ_WRITE,  16*1024*1024);
    CREATEBUF(BITSTREAM_LEN_V, CL_MEM_WRITE_ONLY, EncodedFrameLengthSize * sizeof(unsigned long long));
    CREATEBUF(DUMMY_CNT, CL_MEM_WRITE_ONLY, dummy_cnt * sizeof(unsigned long long));

    //
    // Send Kernel Parameters
    //
    OCLCHECK(clSetKernelArg, kernel, 0, sizeof(cl_mem),       &INPUT_FP_Y_V);
    OCLCHECK(clSetKernelArg, kernel, 1, sizeof(cl_mem),       &INPUT_FP_U_V);
    OCLCHECK(clSetKernelArg, kernel, 2, sizeof(cl_mem),       &INPUT_FP_V_V);
    OCLCHECK(clSetKernelArg, kernel, 3, sizeof(unsigned int), &FRAME_WIDTH_V);
    OCLCHECK(clSetKernelArg, kernel, 4, sizeof(unsigned int), &FRAME_HEIGHT_V);
    OCLCHECK(clSetKernelArg, kernel, 5, sizeof(unsigned int), &FIXED_QP_V);
    OCLCHECK(clSetKernelArg, kernel, 6, sizeof(unsigned int), &BITRATE_V);
    OCLCHECK(clSetKernelArg, kernel, 7, sizeof(unsigned int), &INTRA_PERIOD_V);
    OCLCHECK(clSetKernelArg, kernel, 8, sizeof(unsigned int), &DUMMY_OUTRATIO);
    OCLCHECK(clSetKernelArg, kernel, 9, sizeof(unsigned int), &DUMMY_DELAY);
    OCLCHECK(clSetKernelArg, kernel, 10, sizeof(cl_mem),       &BITSTREAM_BASE_V);
    OCLCHECK(clSetKernelArg, kernel, 11, sizeof(cl_mem),       &REF_BASE_V);
    OCLCHECK(clSetKernelArg, kernel, 12, sizeof(cl_mem),       &BITSTREAM_LEN_V);
    OCLCHECK(clSetKernelArg, kernel, 13, sizeof(cl_mem),       &DUMMY_CNT);

    // ----- OpenCL Initialization, Buffer Setup, Kernel Argument Setting ----
```

After calling the general OpenCL context initialization, this block of
code performs two operations. First, it allocates communication
buffers and associates them with the static global buffer
variables. This is performed through the **CREATEBUF** macro
calls. The macro is defined in the **xlnx_copy_config.h** header file
and calls **clCreateBuffer**.  

Second, the function calls the **OCLCHECK** macro. This macro is used
to hide the error checking and simply calls **clSetKernelArg**. With
each call to this method a static arguments for the kernel execution
is set.__


Now we can move on to the actual execution function
(**ff_copy_sw_xlnx**). Here we start by deleting everything from the
function header all the way to the comment lines (**// -----**). In
between the comment lines you should add:

``` C
    // ----- Execute the HW kernel -------------------------------------------

    size_t global_work_size_1d[1] = {1};
    size_t local_work_size_1d[1] = {1};
    unsigned int* dcnt =0;
    dcnt = malloc(sizeof(unsigned int));

    // Setup input buffer with new data
    OCLCHECK1(clEnqueueWriteBuffer, cl_env->command_queue, INPUT_FP_Y_V, CL_TRUE, 0, CurSourceImageSize[Y] * s
izeof(unsigned char), pic->data[Y], 0, NULL, NULL);
    OCLCHECK1(clEnqueueWriteBuffer, cl_env->command_queue, INPUT_FP_U_V, CL_TRUE, 0, CurSourceImageSize[U] * s
izeof(unsigned char), pic->data[U], 0, NULL, NULL);
    OCLCHECK1(clEnqueueWriteBuffer, cl_env->command_queue, INPUT_FP_V_V, CL_TRUE, 0, CurSourceImageSize[V] * s
izeof(unsigned char), pic->data[V], 0, NULL, NULL);

    // Run kernel
    OCLCHECK1(clEnqueueNDRangeKernel, cl_env->command_queue, kernel, 1, NULL, global_work_size_1d, local_work_
size_1d, 0, NULL, NULL);

    clFinish(cl_env->command_queue);

    //
    // Read the result from output buffer
    // Get the length of the Encoded Frame first as this varies!
    //
    OCLCHECK1(clEnqueueReadBuffer, cl_env->command_queue, BITSTREAM_LEN_V, CL_TRUE, 0, EncodedFrameLengthSize 
* sizeof(unsigned int), dstSize, 0, NULL, NULL);
    OCLCHECK1(clEnqueueReadBuffer, cl_env->command_queue, BITSTREAM_BASE_V, CL_TRUE, 0, (*dstSize) * sizeof(un
signed char), dst, 0, NULL, NULL);
    OCLCHECK1(clEnqueueReadBuffer, cl_env->command_queue, DUMMY_CNT, CL_TRUE, 0, sizeof(unsigned int), dcnt, 0
, NULL, NULL);

    free (dcnt);
    // ----- Execute the HW kernel -------------------------------------------

```

Each time this function is executed, the input data buffers are filled
(**clEnqueueWriteBuffer**) and the kernel is requested to be executed
(**clEnqueueNDRangeKernel**). Once execution is completed, checked by
**clFinish**, the output data is read back from the FPGA to the host 
(**clEnqueueReadBuffer**).

Once all the frames in the input stream are completed, the function
associated with the close function pointer is called
(**ff_uninit_copy_sw_xlnx**). 

``` C
    // ----- Release OpenCl references ---------------------------------------
    clReleaseKernel(kernel);
    clReleaseProgram(program);
    clReleaseMemObject(INPUT_FP_Y_V);
    clReleaseMemObject(INPUT_FP_U_V);
    clReleaseMemObject(INPUT_FP_V_V);
    clReleaseMemObject(BITSTREAM_BASE_V);
    clReleaseMemObject(BITSTREAM_LEN_V);
    clReleaseMemObject(DUMMY_CNT);
    // ----- Release OpenCl references ---------------------------------------
```

Each call in this block is related to cleaning up the global static
OpenCL references. Calling these functions is critical in case of
multi file processing to avoid memory leaks.  

This completes the code modifications and we can go back to building
the executable.

``` shell
cd ~/ffmpeg
./build_ffmpeg.sh
```

Once completed the ffmpeg executable can be exercised with the help of
the same command as we previously executed the software version.

``` shell
sudo ./run.sh cp_sw.cmd
```

Once again, the results can be verified by comparing the input and outputs.

``` shell
diff sw_outdir/bbb_sunflower_1k1c_out.yuv ../vectors/bbb_sunflower_1080p_30fps_normal_1000frames.yuv
```

Before we complete this part of the tutorial, I would like to mention
that nowhere in this exercise have we been looking at the actual code
of the accelerator. This part is completely hidden through the
awsxclbin file and the access rights to the AFI. This is important to
note, as this allows accelerated algorithms to be integrated into
various frameworks and completely hiding the IP of the implementation.

<a name="HEVCAcceleration"></a>
## FFMPEG HEVC Acceleration

The second part of this tutorial is simply exercising two
implementations of an HEVC algorithm. Both versions are already
compiled and integrated with the FFMPEG framework we have build in the
first part.

The first implementation is a pure software implementation. To execute
this software version perform the following steps.

``` shell
cd ~/ffmpeg
sudo ./run.sh hevc_sw.cmd
```

As you can see this results in a framerate of roughly 9 fps and a
total runtime of 55 seconds.

Now the second implementation exercises a hardware accelerated
implementation. As previously noted, the hardware is capable to
process multiple channels on multiple kernels in parallel such that we
had to limit the accelerated implementation to utilize only a single
kernel to achieve an apples to apples comparison. To execute the
accelerated hardware please run:

``` shell
cd ~/ffmpeg
sudo ./run.sh hevc_hw.cmd
sudo ./run.sh hevc_hw.cmd
```

The first run should report around 30 fps and each following call
around 50 fps. The difference is that in the first call, the FPGA is
actually getting loaded with the code. Once loaded, each following run
is checking the loaded image and skipping the reload if possible.

This apples to apples comparison exhibits a 5.5x speed
improvement. However, when fully deployed, the hardware accelerated
implementation can encode several channels with several kernels in
parallel, making it extremely valuable for content providers
requiring different streams of data encoded for different media
devices.
