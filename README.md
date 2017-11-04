# Xilinx Developer Lab - SC17

Welcome to the Xilinx developer lab at SC17.
During this session you will gain hands-on experience with AWS accelerated applications leveraging FPGA devices.

Let's get started...

## 1. Getting Setup

In this initial section, you will access a pre-configured EC2 F1 instance. You will "remote desktop" to that instance. You will then load the Lab design files and associated instructions.
[Setting up the session](Setup.md).

## 2. Running "hello world"

Running a basic vector addition example showcases the main phases of verification with both software and hardware emulation.
Finally running the example's kernel on the actual hardware to confirm the output matches the emulation results.
Please note that in order to save time, the AFI (Amazon FPGA Image) was created in advance. 
Link to the [Hello World Lab Instructions](Hello_World_Lab.md)

## 3. The IDCT Lab

The Discrete Cosine Transform (DCT) and its inverse (IDCT) transform time domain signals or images into a spatial domain. This  transformation allows very efficient compression algorithms used in most video codec.
[IDCT Lab Instructions](IDCT_Lab.md)

## 4. The FFMPEG Lab

The FFMPEG framework provides the infrastructure and implementation to effectively work with a wide variety of codecs. This framework is introduced in this tutorial and it is shown how to effectively integrate hardware accelerated algorithms. Please refer to the [FFMPEG Lab Instructions](FFMPEG_Lab.md) for more details.

