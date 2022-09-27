# Pick up the vitis version library by tcl command on Windows and Linux OS

## 背景：
本文给想直接使用Vitis HLS 工具在 Standalone 模式下调用Xilinx Vision Library L1 API 的小伙伴提供一个非常容易上手的脚本文件。Vitis Vision库的安装路径有关，如今Vitis HLS 2020.1之后的版本都不再提供OpenCV的预编译库，就更需要开发者们将各自工作环境中的库路径，环境变量都设置好。希望这篇博文能给大家调用Vitis Vision ibrary提供向导，提升效率。

Vitis Vision库是Xilinx官方将Opencv功能转换至易于在FPGA中部署的视觉加速库，可在Vitis环境中实施。其中Vitis Vision库的L1目录提供了在Vitis HLS层级部署的应用实例设计。这个实例设计中C-sim的流程中需要调用OpenCV用于测试平台功能，因此需要现有的OpenCV安装。
为了适应各种用户环境，从2020.1版本开始，Xilinx不再提供带有Vivado / Vitis工具的OpenCV的预安装版本。尽管Vitis在综合布局布线Vision库的流程中不需要OpenCV，但是运行示例设计仿真是必需的。

文章使用 Vitis 2022.1 版本介绍了如何创建独立的Vitis HLS TCL文件，用户只要在将该tcl脚本拷贝在Vision Lirary的实例目录中，即可在命令行模式下跑完Vitis_HLS C仿真，综合，联合仿真以及导出IP等全部流程。
Vision的官方文档中包含GUI用户界面和TCL指令调用Vitis Vision库的教程，该信息位于以下位置：
https://xilinx.github.io/Vitis_Libraries/vision/2022.1/overview.html#getting-started-with-hls
https://xilinx.github.io/Vitis_Libraries/vision/2022.1/index.html#

要利用示例设计或在用户测试平台中引用OpenCV库，必须执行以下步骤：
- 安装OpenCV工具版本3.x
  OpenCV在Linux的安装和环境设置请参考附录A, 在Windows环境下建议使用Mingw编译Opencv安装包。
- 设置环境变量以引用OpenCV安装路径
- 下载Vitis Vision library
- 创建TCL脚本 并在Vitis HLS命令行执行

注意：2022.1 Vitis Vision库已使用OpenCV库的4.4.0版进行了验证。比该版本更新的任何版本都可以使用，但是，版本4.x可能相对于3.x版本具有库功能更改，可能需要修改示例设计测试平台。因此，建议使用OpenCV 4.x版运行示例设计。OpenCV库仅提供测试平台功能，不是必需的，并且不会以任何方式影响Vision内核的实现。

## 环境设置：
	
L1内核github页面的以下位置描述了加速平台makefile流的环境设置要求：
https://github.com/Xilinx/Vitis_Libraries/tree/master/vision/L1

Linux 环境变量设置要求：
```
source < path-to-Vitis-installation-directory >/settings64.sh
source < part-to-XRT-installation-directory >/setup.sh
export DEVICE=< path-to-platform-directory >/< platform >.xpfm
export OPENCV_INCLUDE=< path-to-opencv-include-folder >
export OPENCV_LIB=< path-to-opencv-lib-folder >
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:< path-to-opencv-lib-folder >
```

Windows环境变量设置要求：
示例如下所示，并且每个用户的设置会有所不同，具体取决于OpenCV和编译器工具的安装目录。


注意：必须在用户的环境中正确设置LD_LIBRARY_PATH动态库的搜索路径环境变量和OpenCV PATH信息，此脚本和Vitis Vision示例设计才能正常工作。此外，OpenCV的包含库和二进制文件的路径必须包含在系统的环境变量中。否则，将导致仿真期间库包含错误。

## 操作步骤
	
要运行Vitis HLS tcl脚本，请执行以下操作：
-将修改好的tcl 脚本放在<path> / vision / L1 / example / resize目录中
-打开Vitis HLS命令行外壳并 cd <path> / vision / L1 / example / resize目录
-运行以下命令：vitis_​​hls -f run_hls_standalone.tcl

## Vitis HLS TCL脚本详细解释

本文提供了一个TCL脚本，用于在makefile流之外运行L1调整大小（resize）示例设计。该脚本基于以下环境设置：
-OpenCV版本3.4.11
Linux:
set XF_PROJ_ROOT "/home/vicky/Xilinx/Vitis_Libraries-master/vision" 	
set OPENCV_INCLUDE "/home/vicky/opencv/include" 
set OPENCV_LIB "/home/vicky/opencv/lib"

Windows：
- OpenCV include directory : C:/Data/OpenCV/build_win64/install/include
- OpenCV library directory : C:/Data/OpenCV/build_win64/install/x64/mingw/lib
- Vitis Vision Directory : C:/Data/Vitis_Libraries/Vitis_Libraries-master/vision/

TCL脚本文件包含以下部分，本博文将逐一介绍：

	• 代表OpenCV和项目环境的变量声明
	• 项目创建命令
	• 使用Vitis Vision库添加设计文件包括路径
	• 使用OpenCV和Vitis Vision库添加Testbench文件包括路径
	• 使用OpenCV链接器参考进行C仿真
	• Vitis HLS IP合成
	• 具有OpenCV链接器参考的RTL协同仿真
	• 封装导出IP

1. 变量声明：

变量声明部分的第一部分声明了一些变量，这些变量复制makefile流和该流生成的settings.tcl文件的环境变量。这些变量指向Vitis Vision Includes，OpenCV头文件和OpenCV预编译的库。这些位置可能会根据用户系统的安装路径而有所不同。
```
set XF_PROJ_ROOT“ C：/ Data / Vitis_​​Libraries / Vitis_​​Libraries-master / vision”   ＃Vitis Vision库的包含目录
set OPENCV_INCLUDE“ C：/ Data / OpenCV / build_win64 / install / include”         ＃OpenCV头文件目录
set OPENCV_LIB“ C：/ Data / OpenCV / build_win64 / install / x64 / mingw / lib”  ＃OpenCV编译的库目录
```

下一个变量声明部分有助于创建Vitis HLS项目，并有助于使脚本可移植：
```
set PROJ_DIR“ $ XF_PROJ_ROOT / L1 / examples / resize”
set SOURCE_DIR“ $ PROJ_DIR /”
set PROJ_NAME“ hls_example”
set PROJ_TOP“ resize_accel”
set SOLUTION_NAME“ sol1”
set SOLUTION_PART“ xcvu11p-flgb2104-1-e”
set SOLUTION_CLKP 5
```

最后，最后一部分声明变量，这些变量表示HLS引用和使用库所需的引用路径和标志。这里我们发现在一个易用性高的脚本中，使用变量而不是代码有助于理解如何使用这些选项。
```
set VISION_INC_FLAGS“ -I $ XF_PROJ_ROOT / L1 / include -std = c ++ 0x” ＃Vitis Vision包含路径和C ++ 11设置
set OPENCV_INC_FLAGS“ -I $ OPENCV_INCLUDE” ＃OpenCV包含目录引用
set OPENCV_LIB_FLAGS“ -L $ OPENCV_LIB” ＃OpenCV库参考
```

**注意：**
在Windows中，库引用必须包含版本号。本示例使用OpenCV 4.4.11安装。精确的包含格式将取决于用户的安装，并且可能与下面列出的格式不同。
```
set OPENCV_LIB_REF“ -lopencv_imgcodecs3411 -lopencv_imgproc3411 -lopencv_core3411 -lopencv_highgui3411 -lopencv_flann3411 -lopencv_features2d3411”
```
在Linux include语句不使用版本号，并给出如下：
```
set OPENCV_LIB_REF“ -lopencv_imgcodecs -lopencv_imgproc -lopencv_core -lopencv_highgui -lopencv_flann -lopencv_features2d”
```
2. 项目创建：

项目创建部分非常简单，它会创建一个新的项目目录和项目文件：

open_project -reset $PROJ_NAME

添加设计文件：

- 引用单个HLS内核文件：add_files“ $ {PROJ_DIR} /xf_resize_accel.cpp”
- 引用Vision库和特定于项目的包含合成目录：-cflags“ $ {VISION_INC_FLAGS} -I $ {PROJ_DIR} / build
- 引用了用于C仿真的Vision库和特定于项目的包含目录：-csimflags“ $ {VISION_INC_FLAGS} -I $ {PROJ_DIR} / build”
 
完整的命令如下所示：
```
add_files“ $ {PROJ_DIR} /xf_resize_accel.cpp” -cflags“ $ {VISION_INC_FLAGS} -I $ {PROJ_DIR} / build” -csimflags“ $ {VISION_INC_FLAGS} -I $ {PROJ_DIR} / build”
```

添加Testbench文件包括：

- 引用Test bench文件：add_files -tb“ $ {PROJ_DIR} /xf_resize_tb.cpp”
- 引用Vision库和特定于项目的包含目录：-cflags“ $ {OPENCV_INC_FLAGS} $ {VISION_INC_FLAGS} -I $ {PROJ_DIR} / build”
- 引用Vision库和特定于项目的C仿真目录：-csimflags“ $ {OPENCV_INC_FLAGS} $ {VISION_INC_FLAGS} - I $ {PROJ_DIR} / build”

请注意，在测试台标志和设计文件标志中添加了$ {VISION_INC_FLAGS}变量。此设置引用OpenCV包含文件。
 
完整的命令如下所示：
```
add_files -tb“ $ {PROJ_DIR} /xf_resize_tb.cpp” -cflags“ $ {OPENCV_INC_FLAGS} $ {VISION_INC_FLAGS} -I $ {PROJ_DIR} / build” -csimflags“ $ {OPENCV_INC_FLAGS} $ {PROSION_IN /建立”
```
3. 项目设置：

现在已经添加了所有需要的C源文件，执行项目创建的最后一步。这些命令设置HLS IP的顶层函数，并创建一个所需的项目solution.
```
set_top $ PROJ_TOP＃设置HLS IP的顶级文件
open_solution -reset $ SOLUTION_NAME＃创建项目解决方案
set_part $ SOLUTION_PART＃设置解决方案部分
create_clock -period $ SOLUTION_CLKP＃设置项目目标时钟周期
```
4. Run c-sim：

本部分通过将HLS IP和Testbench设计发送给编译器进行编译和执行，来执行HLS流的C仿真阶段。此命令用于设置编译器链接器标志和testbench文件，以及：

- 引用OpenCV包含和预编译的库目录：-ldflags“ -L $ {OPENCV_LIB} $ {OPENCV_LIB_REF}”
- 包括用于验证测试台的图像作为主要功能的参数：-argv“ $ {XF_PROJ_ROOT} /data/128x128.png”
 
完整的命令如下所示：
```
csim_design -ldflags“ -L $ {OPENCV_LIB} $ {OPENCV_LIB_REF}” -argv“ $ {XF_PROJ_ROOT} /data/128x128.png”
```

5. C到RTL综合：

本部分执行Vitis HLS C到RTL综合阶段。此阶段暂时不添加其他的标志或选项。

csynth_design

6. C/RTL协同仿真：

本部分在合成后执行Vitis HLS IP的RTL协同仿真。HLS会自动根据C test bench生成RTL testbench 进行协同仿真，以下指令用于设置编译器链接器标志和testbench文件，以及：

- 引用OpenCV包含和预编译的库目录：-ldflags“ -L $ {OPENCV_LIB} $ {OPENCV_LIB_REF}”
- 包括用于验证测试平台的图像：-argv“ $ {XF_PROJ_ROOT} /data/128x128.png”
 
完整的命令如下所示：
```
cosim_design -ldflags“ -L $ {OPENCV_LIB} $ {OPENCV_LIB_REF}” -argv“ $ {XF_PROJ_ROOT} /data/128x128.png”
```
7. 导出IP：
Vitis HLS流程的最后阶段是设计的输出。本示例导出RTL的设计并运行Vivado Synthesis，以获取准确的资源利用率和估计的时序结果。
```
export_design -flow syn -rtl verilog
```
注意：导出RTL的设计并运行Vivado Synthesis进行布局布线的过程需要在Vivado工具中先载入有效的license

附件为在Ubuntu 18.04 版本在2022.1 上运行成功的tcl shell, 大家可以下载后稍作修改，根据本文流程在自己的环境中进行实验，有问题欢迎在本帖下方留言。


### 附录A -- 下载并安装OpenCV on Linux

1. 打开一个终端, 先不要执行任何有关 Vivado 或 Vitis 的set up 指令, 否则 CMake 会失败.
2. 按照自己的意愿在Home 创建一个home目录下的opencv目录 –> for instance:  cd ~/src_opencv
3. git clone --branch 4.4.0 https://github.com/opencv/opencv.git

creates sub-folder “opencv” with the source files –> ~/src_opencv/opencv

4. git clone --branch 4.4.0 https://github.com/opencv/opencv_contrib.git

creates sub-folder “opencv_contrib” with the source files –> ~/src_opencv/opencv_contrib

5. mkdir build  ->  this will create a sub-folder “build”  -> ~/src_opencv/build
6. cd build
7.跑以下指令去生成编译 OpenCV library 的makefile:

cmake -D CMAKE_BUILD_TYPE=Release -OPENCV_EXTRA_MODULES_PATH=~/src_opencv/opencv_contrib/modules -D CMAKE_INSTALL_PREFIX=~/opencv ~/src_opencv/opencv

8. From the “build” directory 输入以下两个指令

make -j7
make install

9. 编译安装完成后，你就会看到Opencv库在以下路径中 ~/opencv
