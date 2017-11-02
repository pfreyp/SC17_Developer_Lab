
# IDCT example Tutorial on AWS-F1

## Contents
1. [Introduction](#Introduction)
1. [Prerequisite](#Prerequisite)
1. [Setup](#Setup)
1. [Run Emulations](#Emulation)
1. [Analyzing the Reports](#Analyzing)
1. [Optimization](#Optimization)
1. [Running on F1](#Running)
1. [Conclusion](#Conclusion)
1. [Reference](#Reference)

<a name="Introduction"></a>
## Introduction  
The following tutorial is to help teach the basics of the SDx IDE as well as the SDAccel development process by going through creating a project, running software and hardware emulation, and analyzing various reports generated from the emulations in order to help identify how to optimize code. At the conclusion of the tutorial, there are various links to different resources to help provide details on aspects of optimizing kernel code for projects.

The design is a Inverse Discrete Cosine Transform, which is used heavily in audio/image processing and is based off of the Fourier Transform. Please refer to the Wikipedia article  [Descrete Cosine Transform](https://en.wikipedia.org/wiki/Discrete_cosine_transform) for more information. 

 
<a name="Prerequisite"></a>
## Prerequisite 
This example assumes that the aws-fpga git is cloned into directory ~/aws-fpga and the actual design files krnl_idct.cpp and idct.cpp is present in ~/idct/src/. If any of these directories are different in your setup, please replace any reference in the tutorial to point to your installation directories. 

In addition, the user should have logged into his/her AWS-F1 instance and bring up graphical desktop environment. The following steps assumes as a starting point an active terminal on the graphical desktop running the default “bash”-shell

1. Before the SDAccel tool suite can be started the environment needs to be configured. Towards this end, change into the ~/aws-fpga directory and source “sdaccel_setup.sh” <br>
    ```
    cd ~/aws-fpga
    source sdaccel_setup.sh
    ```
    The setup might create some warning messages regarding missing libraries but these can be safely ignored.

1. Launch SDx with the sdx command. 
    ```
    sdx
    ```
    The **Eclipse Launcher** will start up first with the workspace directory menu. It allows you to choose where you want the location of your workspace (and projects) to go. For this example, I suggest to create a workspace in your root directory or next to your idct directory. This is done through the following steps: **Browse… -&gt; Create Folder -&gt; name it “workspace” &lt;Enter&gt; -&gt; OK**. The Workspace path should read &lt;home&gt;/workspace. **OK** this selection.  
![](./figures/Welcome.PNG)

1. The SDx Welcome screen should be visible. Next we will need to get the Amazon F1 Platform recognized by the project. This requires the addition of a custom platform. On the Welcome screen: Select **Add Custom Platform -&gt; Add CustomPlatform… -&gt;** navigate to **~/aws-fpga/SDAccel/aws_platform/Xilinx_aws-vu9p-f1_4ddr-xpr-2pr_4_0 -&gt; OK**.
Press **OK** again in the Hardware Platform Repositories form.  
![](./figures/Platform.PNG)
 
1. Now the initial setup for an AWS-F1 project is completed. Next we can create a new project through the **Create SDx Project** button on the Welcome screen.  
 
1. In the **Project name** field of the **New Project** window type **IDCT**, and click **Next**.  
![](./figures/Project.PNG)

1. In the **Choose Hardware Platform* window select the **AWS-VU9P (4ddr-xpr-2pr) (custom)** platform. Press **Next**.

1. The **Software Platform** window only has **Linux on x86** and **OpenCL** as valid options, click **Next**. 

1. The **Templates** window shows all the downloaded Github examples. However, in this tutorial we will are not starting with any of the installed templates, but select **Empty Application** and click **Finish**.  
![](./figures/Template.PNG)

1. We have now successfully created a new SDAccel project called IDCT for the AWS-VU9P-F1 platform. This is prominently displayed in the SDx Project Settings window in the center of the GUI. Let us briefly tour the sections of the default GUI layout in more detail:
    * The main menu bar is located on the top. This menu bar allows direct access to all general project setup and GUI window management tasks. As most tasks are performed through the different setup windows, the main menu is mostly used to recover from accidently closed windows or to access the tool help.
    * Just below the main menu bar is the SDAccel Tool Bar located.  This toolbar provides access to the most common tasks in a project. From right to left these are File Management functions (new, save, save all), Configuration Management, Build, Build All, start Debug, and Run. Most buttons have a default behavior as well as pulldowns.
    * The Project Explorer is the top window on the left hand side. The explorer window is designed to manage and navigate through project files. 
    * In the middle is the SDx Project Settings window. This window is intended for the project management and presents the key aspects of an SDx Project. 
    * The Outline window on the right hand side helps in the navigation of files. The content of the outline depends on the file currently selected in the main window.
    * In the left bottom section we have the Reports window. The Reports window allows easy access to all reports generated by SDAccel. 
    * The remaining windows along the bottom of the main window, accommodate the various consoles and terminals which contain the output of the individual SDAccel executables. A typical examples are compilation errors or the tool output when running.  
![](./figures/SDxWindows.PNG)
 
<a name="Setup"></a>
## Setup the Project

In this section, we are going to setup SDAccel. More precisely, we will show how to import source files from the local file system to the project, determine hardware functions, and generally configure the SDAccel flow. 

1. From the **Project Explorer** window, select the **src** directory, right-click and select **Import…**  
![](./figures/Import.PNG)

1. In the **Import** window, navigate to the **General &gt; File System** and click **Next**.  
![](./figures/FileSystemImport.PNG)

1. In the next window, press the Browse button and navigate to where the **IDCT** source files are located (~/idct/src). Press **Okay** to accept the source directory. After that, the file system will show the two files contained in the directory. Select both files and press **Finish**.  
![](./figures/FileSelectImport.PNG)

1. To verify that both files were included in the project, we can use the **Project Explorer**. Expand the **src** file to make sure all files got added.  
![](./figures/ProjectExplorer.PNG)

1. Next, we need to identify the functions to be implemented in hardware (aka accelerators / kernels). This is performed through the **SDx Project Settings** window. As we have entered the source files, sdx will help us to identify the accelerators. Locate the **Add Hardware function…** button: ![](./figures/AddHW.PNG) and click it. The **Add Hardware Functions** window pops up and will analyze the source code provided. It identifies any functions that are possible for being a hardware function. Select the **krnl_idct** function and click **OK**.  
![](./figures/KrnlSelect.PNG)

1. The **Hardware Functions** section now contains a binary container that includes the **krnl_idct** function.   
![](./figures/ProjectWindowWithKrnl.PNG)

1. Next, we need to take care of the fact that the kernel and host are configured to utilize two different DDR memories one for reading and one for writing of the data. This mapping of AXI master ports needs to be set during the link stage of the kernel compilation step. Towards that end, select **Properties** from the **Project** menu in **Main Menu** bar. Expand the **C/C++ Build** menu item and select **Settings** from the submenu.  

    ![](./figures/AllConfigurations.PNG)  

    First we want to select **[All configurations]** from the Configuration dropdown menu. This will ensure that the mapping is applied to all build configurations. Next, we select the **Tool Settings** tab and navigate to the **SDx XOCC Kernel Linker -&gt; Miscellaneous** section. On the right hand side, it is now possible to specify additional link options such as the AXI master mapping commands. Please add the following three options in this window (one line) and hit **Okay**.
    ```
    --xp misc:map_connect=add.kernel.krnl_idct_1.M_AXI_GMEM.core.OCL_REGION_0.M00_AXI 
    --xp misc:map_connect=add.kernel.krnl_idct_1.M_AXI_GMEM1.core.OCL_REGION_0.M00_AXI 
    --xp misc:map_connect=add.kernel.krnl_idct_1.M_AXI_GMEM2.core.OCL_REGION_0.M01_AXI
    ```
    More information regarding the mapping command can be found in the [UG1023 SDAccel Environment User Guide](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_2/ug1023-sdaccel-user-guide.pdf).  
![](./figures/XOCCKrnlLinker.PNG)

1. For the purpose of this training, we are also going to overwrite some default SDx behavior. For this purpose, we add the absolute path of the HLS configuration file to the Kernel Compiler. Select **Properties** from the **Project** menu in **Main Menu** bar, expand the **C/C++ Build** menu item and select **Settings** from the submenu.

    Again, we want to select **[All configurations]** from the Configuration dropdown menu to ensure that the mapping is applied to all build configurations. Next, we select the **Tool Settings** tab and navigate to the **SDx XOCC Kernel Compiler -&gt; Miscellaneous** section. On the right hand side, it is now possible to specify additional compile options. Please add the following and confirm by hitting **Okay**.
    ```
    --xp prop:solution.hls_pre_tcl=~/idct/support/config.tcl
    ```  
    ![](./figures/XOCCKrnlCompiler.PNG)

1. Finally, this design is setup to receive the filename of the bitstream as command line argument. Especially during initial design flow were the designers might often switch between emulation modes, this is a simple way to avoid host code modifications based on build configuration. From the **Main Menu**, pick **Project** and select **Properties** and click on the **Run/Debug Settings** menu item. Select the **IDCT-Default** configuration and click **Edit…**.   

    ![](./figures/EditRunConfig.PNG)  

    In the **(x)= Arguments** tab check the **Automatic add binary container(s) to arguments** option, which will add ../binary_container_1.xclbin to the program argument list. Press **OK** and again **OK** in the Properties for IDCT window and we are done configuring the runtime environment.  

    ![](./figures/AutomaticAddBinaryCont.PNG)

<a name="Emulation"></a>
## Run Emulations (Software/Hardware)  

In this section, we demonstrate the software/hardware emulation flow. 

1. Now, the project is completely setup and ready for compilation and emulation. The SDx environment uses Makefiles to perform incremental compilation of the project. This means that unless file changes require the recompilation of the host code or of the kernel code compilation no compilation is performed.

    In the upper right corner of the **SDx Project Settings** window, the current configuration is shown. Please ensure that **“Emulation-CPU”** is selected. CPU-Emulation is intended to verify the algorithm and identify syntax issues through pure C/C++ emulation. To actually run software emulation for the design, click the **Run** button, ![](./figures/RunButton.PNG). This will build both the kernel and host code and execute emulation.

    You will notice that the **Console** will have a detailed build log of compiling the kernel first and then the host code. Once the build is complete, an output console will be present for any standard output the application produces. 

1. Once the software emulation is complete, in the **SDx Project Settings** window, click the dropdown **Emulation-CPU**, and select **Emulation-HW**. Now the project is set for running a more detailed emulation compared to the software emulation. In this emulation mode the actual Register Transfer Logic (RTL) generated from the C/C++ is simulated and run together with the host code application. Go ahead and click the same **Run** button again.

    The runtime of the Hardware emulation will take longer than the Software Emulation because the kernel code is being compiled to create the more detailed simulation models and linked with the platform. In addition, the more detailed simulation models require more simulation time. However, this will allow to produce more accurate reports on kernel performance in comparison to software emulation. This step should be used in helping identifying performance issues of the design.

    Next will be analyzing the reports produced by both analysis runs. 

<a name="Analyzing"></a>
## Analyzing the Reports  

In this section we will cover how-to locate and read the various reports generated by the emulation runs. The goal of the section is to understand the analysis reports of SDAccel in this section before we utilize them extensively in the next section.  
![](./figures/Reports.PNG)

1. In the **Reports** window you will see a tree layout of folders and reports for all runs and open projects. Therefore on the top level we see wee the **IDCT** project in which we have executed two runs, namely **Emulation-CPU** and **Emulation-HW**.

    If we expand the **Emulation-CPU** folder, we see the **IDCT-Default** run configuration. The run, of this configuration created a **Profile Summary** and **Application Timeline** report. 

    Similarly, the **Emulation-HW** folder contains all reports created with this build configuration. However, as this configuration performs High-Level Synthesis under the hood, first estimates regarding the performance of the actual hardware implementation are available.  These are summarized in the **System Estimate** and accessible in more in all detail from the **HLS Report** located in the **binary_container_1 -&gt; krnl_idct** subhierarchy. Similar to the emulation-cpu flow, the run configuration **IDCT-Default** produced a **Profile Summary** and an **Application Timeline** report.

1. Let us first look at the **Emulation-CPU Profile Summary**. Double click the **Profile Summary** to open up the report.   
    ![](./figures/SWProfile.PNG) 

    This report provides the most data related to how the application runs. You will notice that the report has four tabs: **Top Operations**, **Kernels & Compute Units**, **Data Transfers**, and **OpenCL APIs**. Click through each one and become familiar with what each tab data shows.

    **Top Operations**: This tab contains all the major top operations of memory transfer between host and kernel to global memory, and kernel execution. This allows you to identify throughput bottlenecks when transferring data. Being efficient at transferring a lot of data to the kernel/host allows for faster execution times.
    **Kernels & Compute Units**: Contains the number of times the kernel was executed with the total, minimum, average, and maximum run times. If the design has multiple compute units, it will show each compute unit’s utilization. When accelerating an algorithm, the faster the kernel executes the better throughput can be achieved. It is best to optimize the kernel to be as fast as it can be with the data it requires.
    **Data Transfers**: This tab has no bearing in software emulation as no actual data transfers are emulated across the host to the platform. In hardware emulation, this will show the throughput and bandwidth of the read/writes to the global memory that the host and kernel share.
    **OpenCL APIs**: Shows all the OpenCL API command executions, how many time each was executed, and how long they take to execute.

1. Now let us examine the **Application Timeline** report by double clicking it in the **Emulation-CPU** hierarchy. This shows a visual representation of the **OpenCL APIs** from the **Profile Summary** and in what order the APIs execute across the emulation threads.  
    ![](./figures/SWTimeline.PNG)

1. Now let us move on to the **Emulation-HW** reports. We start by double clicking on the **HLS Report**. This report is specific to the kernel code, and how it translates to the platform. It contains information about clocking, resources, device utilization, and more. For the purpose of this tutorial, the only section of this report to pay attention to is the **Performance Estimates**. This section will provide information on the latency of the kernel, as well as loop implementation details.  
    ![](./figures/HLSReport.PNG)

1. Let us close all reports now before moving on to visualizing two reports side by side. Press **X** on the tabs.

1. Next we will open the **Profile Summary** of the **Emulation-HW** run and drag it to the right (next to the Outline) such that the center window can be used to show the **Profile Summary** of the **Emulation-CPU** run. This allows us to directly compare the two reports. Based on your screen, it is probably best to maximize the application window and minimize the Project Explorer and Reports window to have the maximum information displayed side by side.  
    ![](./figures/ProfileSummaryComp.PNG)
 
1. Notice that in comparison, the hardware emulation has far more data as well as a **Profile Rule Checks** section. This section is intended to provide guidance regarding the overall design performance. These profile rules examine the profile data and compare them to general threshold values. If a checks do not meet the threshold value, the right most column provides guidance on what can be done to help improve design performance.

1. Click on the **Kernels & Compute** Units tabs of both reports to do a comparison between the software and hardware emulation reports of the kernel execution. Take note that in the software emulation, the total time is significantly larger than in the hardware emulation. This is because with software emulation, the host and kernel are running only on the CPU, while the hardware emulation is running in a simulation mode that emulates what the platform would be executing. This sounds counter intuitive given that the Hardware Emulation wall clock runtime is much larger, however, this longer runtime is due the much greater detail of the simulation run.
 
1. Now let us take some measurements to establish a baseline of the current implementation of the design. Towards that end, extract from the **Profile Summary** report of the **Emulation-CPU** run the following numbers and note them down:

    - Kernel Total Time (ms):
    - Kernel Min Time (ms):	
    - Kernel Max Time (ms):	


    For the Emulation-HW configuration, we record **Kernel Total Time** from the **Profile Summary** and from the **HLS Report -&gt; Performance Estimates** section the latency and interval numbers. 

    - Kernel Total Time (ms):
    - Latency (min/max):
    - Interval (min/max):	

1. This concludes our introduction to the SDAccel GUI Analysis capabilities. In the next section we will focus on some of the performance aspects of the current implementation and how they can be improved.  

<a name="Optimization"></a>
## Optimization   

In the previous section we familiarized ourselves with the SDAccel Performance Analysis capabilities. Now we are going to utilize the analysis capabilities to determine results of kernel optimization procedures. In this example, we will deal with ARRAY_PARTITON and DATAFLOW optimization. However a full list of optimizations can be found in [UG1253 – SDx Pragma Reference Guide - Chapter 4](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_2/ug1253-sdx-pragma-reference.pdf).

1. Open the kernel file **krnl_idct.cpp** by double clicking the file at the bottom in the **Project Explorer**. All the way at the end of the file is the krnl_idct top function which is only used to separate the HLS interface pragmas from the actual kernel definition function (krnl_idct_dataflow) for readability. The krnl_idct_dataflow function contains 4 blocks, two read_blocks are mapping the AXI interface to a data stream, one call to write_blocks function responsible to map from stream to output AXI interface and finally the execute block which contains the actual IDCT computation.

    The execute function performs multiple idct computations per kernel execution. The actual number is provided to the kernel as argument. Each computation is prepended by reading the wide input data streams and filling up the 8x8 matrix, and similar post processed by writing into the wide output streams. Note, the coefficients of the read operation are read only once as they are considered constant for all idct computations.

1. Let us start by looking at the actual data access pattern of the design. You will need to look at the data formatting and see if it allows efficient pipelined execution. Problematic code usually involves arrays as these get mapped to memories. Memories are resources with a limited number of ports which can introduce sequential behavior and artificial latency. 

    Thus let us have a look at **HLS Report**, specifically at the **latency** of the **krnl_idct** in the **Performance Estimates** section. We can observe a large number. Looking at the instance section below, we can identify the main contributor of this latency to be the instance of the **krnl_idct_dataflow** module.  

    ![](./figures/LatencyKrnlIdct.PNG)

    If we open **krnl_idct_dataflow** module (click on it in hierarchy column to the left), we can see that it’s main contributor is execute (**Performance Estimates -&gt; Latency -&gt; Detail -&gt; Instance**).  

    ![](./figures/LatencyKrnlIdctDataflow.PNG)  

    If we look at execute loop details (**Performance Estimates -&gt; Latency -&gt; Detail -&gt; Loop**), we see that the latency is actually due to the main loop of the function. This loop is pipelined but the initiation interval (Number of clock cycles before the next iteration of the loop starts to process data) is 64 and also the latency of an iteration is quite large.  

    ![](./figures/LatencyExecute.PNG)  

    If we take a look at the source code of execute (**Project Explorer -&gt; src -&gt; krnl_idct.cpp**), we can see that there are three local arrays (iiblock[64], iiq[64], and iivoutp[64]) which are used to hold the local input and output values of the idct. As these arrays are relatively small, they can be mapped to registers rather than memory, which makes them accessible concurrently. This is easily performed through the directive #pragma HLS ARRAY_PARTITION. Please uncomment them in the source code, save the file (Ctrl-S), and rebuild the project, ![](./figures/BuildButton.PNG). 

1. If we look at the newly generated **HLS Report**, we will see considerable reduction in overall latency compared to the previously noted values.

    - Latency (min/max):
    - Interval (min/max):

1. This is a very common optimization pattern. As a result for small arrays upto 64 entries, SDAccel is performing this optimization usually automatically. For the purpose of this training, this default behavior was disabled with the help of the config.tcl file included earlier. It is usually not common to have such a configuration file.

1. If we look at the **HLS Report** of **krnl_idct_dataflow** function, we can also see that the latency of each of the functions contained (**Performance Estimates -&gt; Latency -&gt; Detail -&gt; Instance**) within the block are roughly the same. Also if you look at the access pattern of the arrays in execute, read_blocks, and write_blocks, the data is read and written in sequence. This makes these functions a good candidate to benefit from Dataflow optimization.  
    ![](./figures/LatencyKrnlIdctDataflow2.PNG)

1. Let us enable the dataflow optimization. The **pragma DATAFLOW** is already present in the krnl_idct _dataflow function and just needs to be uncommented. Please have a look at the [UG1253](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_2/ug1253-sdx-pragma-reference.pdf) for precise details about this pragma. However in short, it allows each of these functions to execute as independent processes. As a result of this dataflow optimization, the channels between the different processes do not need to buffer the complete dataset anymore but can actually directly stream the data to the next block. The additional buffering can be removed by commenting out the 3 other **pragmas** in the function, defining the FIFO depth of the streams.  Once the code changes are performed, save the file (Ctrl-S), and rebuild the project, ![](./figures/BuildButton.PNG).

1. Now that we have optimized the kernel considerably, let us compare the predicted performance in the Hardware Emulation flow. For this purpose, simply press the run button, ![](./figures/RunButton.PNG).

    - Kernel Total Time (ms):
    - Latency (min/max):
    - Interval (min/max):
 
![](./figures/EndLatencyRTLReport.PNG)


<a name="Running"></a>
## Running on F1  

Now that we have fully optimized version of the design, we are ready to build the actual hardware bitstream. This process however takes too much time for this tutorial. However, from an SDAccel perspective, the bitstream creation would simply require to switch the active build configuration to System and rebuild.

Once the bitstream is available, in the Amazon F1 instance it is necessary to register the kernel with the secure storage system. This is performed with the help of the **create_sdaccel_afi.sh** script. Running this script kicks off the registration process and it's completion can be verified using the following command.

``` shell
aws ec2 describe-fpga-images --fpga-image-ids <AFI ID>
```

You can try it with the preregistered AFI ID for this tutorial: **afi-0d59c0f2d5fe9df1f**. The output of this command should contain:

``` shell
...
"State": {
    "Code": "available"
},
...
```

A good starting point regarding the specifics of the Amazon SDAccel flow is the ![Quick Start Guide to Accelerating your C/C++ application on AWS F1 FPGA Instance with SDAccel](https://github.com/aws/aws-fpga/blob/master/SDAccel/README.md)

The registration process creates also a secure bitstream handle "binary_container_1.awsxclbin". This handle is provided to you in your idct directory (~/idct/xclbin/binary_container_1.awsxclbin). Since there is no difference in the host executable, we can run the hardware emulation idct executable with the secure bitstream handle, executing the algorithm on the FPGA.

``` shell
sudo sh
source /opt/Xilinx/SDx/2017.1.rte/setup.sh
<workspace>/IDCT/Emulation-HW/IDCT.exe ~/idct/xclbin/binary_container_1.awsxclbin
```

<a name="Conclusion"></a>
## Conclusion  

In this Lab, you explored:
* How to setup SDx for a blank project
* How to run the Software and Hardware Emulation for a specific project
* How to read the various reports generated by the different emulation runs with emphasis on key elements used in identifying areas for optimization improvements
* How to use some basic pragmas in the kernel code to increase performance. 
 
<a name="Reference"></a>
## Reference  

[UG902 Vivado Design Suite User Guide: High-Level Synthesis](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_2/ug902-vivado-high-level-synthesis.pdf)  
[UG1023 SDAccel Environment User Guide](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_2/ug1023-sdaccel-user-guide.pdf)  
[UG1207 SDAccel Environment Optimization Guide](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_2/ug1207-sdaccel-optimization-guide.pdf)  
[UG1253 – SDx Pragma Reference Guide](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2017_2/ug1253-sdx-pragma-reference.pdf)
  
  
  
[More references on SDAccel can be found in the Cloud](https://www.xilinx.com/products/design-tools/acceleration-zone.html#knowledgeCenter)


