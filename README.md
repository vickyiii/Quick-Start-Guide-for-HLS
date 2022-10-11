# Quick Start Guide for HLS

This is a series of quick start guide of Vitis HLS tool in Chinese. 
It explains the basic concepts and the most important optimize techniques you need to understand to use the Vitis HLS tool. 

## Outlines:

1. [Overview on High-level synthesis technology](./Overview_on_HLS.md）
2. [Vitis HLS Process Overview](./Vitis_HLS_Process.md)
3. [Design Methodology](./Design_Mathodology.md)
4. [Defining Interface](./Defining_Interface.md)
5. [Fine-grained Function and Loop Level Parallel Optimization | Unroll & Pipeline](./Unroll_Pipeline.md)
6. [Coarse-grained Task level parallel optimization | Dataflow](./Dataflow.md)
7. [Controlling AXI4 Burst Behavior](./AXI_burst.md)


## Reference Materials

To help you quickly get started with the Vitis HLS, you can find tutorials and example applications at the following locations:
1. Vitis HLS User Guide (https://docs.xilinx.com/r/en-US/ug1399-vitis-hls/Getting-Started-with-Vitis-HLS)
The offical Vitis HLS user guide including the getting started part and the mathodology guide. All the details about the tool chain can be founded in this guide.

2. Vitis HLS Introductory Examples (https://github.com/Xilinx/Vitis-HLS-Introductory-Examples)
Hosts many small code examples to demonstrate good design practices, coding guidelines, design pattern for common applications, and most importantly, optimization techniques to maximize application performance. All examples include a README file, and a run_hls.tcl script to help you use the example code.

3. Vitis Accel Examples Repository (https://github.com/Xilinx/Vitis_Accel_Examples)
Contains examples to showcase various features of the Vitis tools and platforms. This repository illustrates specific scenarios related to host code and kernel programming for the Vitis application acceleration development flow, by providing small working examples. The kernel code in these examples can be directly compiled in Vitis HLS.

4. Vitis Application Acceleration Development Flow Tutorials (https://github.com/Xilinx/Vitis-Tutorials)
Provides a number of tutorials that can be worked through to teach specific concepts regarding the tool flow and application development, including the use of Vitis HLS as a standalone application, and in the Vitis bottom up design flow.