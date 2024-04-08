

本项目有意思的地方在于使用了一个分光棱镜来设计出`伪全息显示`的效果。这个小设备总的来说功能比较多，因为搭载了WiFi和蓝牙能力可以实现很多网络应用，在本仓库中给大家提供了一个开发框架以及一些基础功能（天气、粉丝数监视器等），可以基于我的方案继续扩展实现更多应用。

The interesting part of this project is the use of a spectroscopic prism to design a `pseudo-holographic display` effect. This small device is more functional in general, because it is equipped with WiFi and Bluetooth capabilities can realize a lot of network applications, in this repository to provide you with a development framework and some basic functions (weather, fan number monitor, etc.), you can continue to expand based on my program to achieve more applications.

本项目的硬件方案是基于`ESP32PICO-D4`的，乐鑫的一个很实用的MCU芯片，由于采用了SiP封装是的PCBA整板面积能做到一个硬币大小；软件方面主要是基于`lvgl-GUI`库，我移植了ST7789 1.3寸`240x240`分辨率屏幕的显示屏驱动，同时将`MPU6050`作为输入设备通过感应的方式模拟编码器键值来交互。

The hardware program of this project is based on `ESP32PICO-D4`, a very practical MCU chip of Loxin, due to the use of SiP package is the PCBA board area can be done to the size of a coin; software is mainly based on the `lvgl-GUI` library, I ported the ST7789 1.3-inch `240x240` resolution screen display driver, at the same time, the `MPU6050` as an input device through the inductive way to simulate the encoder key to interact. I ported the ST7789 1.3" `240x240` resolution screen display driver, and at the same time, I used the `MPU6050` as an input device to interact with the encoder keys by sensing the encoder keys.

## 1. 硬件打样说明

## 1. Hardware Proofing Description


`Hardware`文件内包含PCB电路：

The `Hardware` file contains the PCB circuitry:


**外壳加工** `3D Model`文件夹目前包含外壳文件：

**Shell Machining** The `3D Model` folder currently contains shell files:



* **Metal Version** ：该建议使用CNC加工制作
* 
* **Metal Version**: This recommendation is made using CNC machining.

* 
  ![](/5.Docs/Images/Holo2.jpg)


  ![](/5.Docs/Images/Holo.jpg)



## 2. 固件编译说明

## 2. Firmware compilation instructions


固件框架主要基于Arduino开发完成，把Firmware/Libraries里面的库安装到Arduino库目录（如果你用的是Arduino IDE的话）即可。

Firmware framework is mainly based on Arduino development is completed, the Firmware/Libraries inside the library installed to the Arduino library directory (if you use the Arduino IDE) can be.


**这里需要修改一个官方库文件才能正常使用：**

**Here's an official library file that needs to be modified to work properly:**



安装ESP32的Arduino支持包，然后在安装的支持包的`esp32\hardware\esp32\1.0.4\libraries\SPI\src\SPI.cpp`文件中，**修改以下代码中的MISO为26**：

Install the Arduino support package for ESP32, and then in the `esp32\hardware\esp32\1.0.4\libraries\SPI\src\SPI.cpp` file of the installed support package, **modify the MISO in the following code to 26**:

    if(sck == -1 && miso == -1 && mosi == -1 && ss == -1) {
        _sck = (_spi_num == VSPI) ? SCK : 14;
        _miso = (_spi_num == VSPI) ? MISO : 12; // 需要改为26
        _mosi = (_spi_num == VSPI) ? MOSI : 13;
        _ss = (_spi_num == VSPI) ? SS : 15;
        
这是因为，硬件上连接屏幕和SD卡分别是用两个硬件SPI，其中HSPI的默认MISO引脚是12，而12在ESP32中是用于上电时设置flash电平的，上电之前上拉会导致芯片无法启动，因此我们将默认的引脚替换为26。

This is because the hardware connection to the screen and the SD card is with two hardware SPIs respectively, where the default MISO pin for the HSPI is 12, and 12 is used in ESP32 to set the flash level during power up, and pulling it up before powering up will result in the chip not being able to boot up, so we replace the default pin with 26.


## 3. Visual Studio模拟器 & 图片转换脚本

## 3. Visual Studio Emulator & Image Conversion Scripts

在`Software`文件夹中包含了一个Visual Studio的工程，用VS打开（需要安装C++开发组件）后可以在电脑上模拟LVGL的界面效果，改好之后代码粘贴到Arduino固件那边就可以完成界面移植。

In the `Software` folder contains a Visual Studio project, open it with VS (you need to install the C++ development component) and then you can simulate the LVGL interface effect on your computer, and then paste the code into the Arduino firmware side to complete the interface porting.

> 这样省得每次修改都要重新交叉编译Arduino的固件，提升开发效率。
> 
> This saves you from having to cross-compile the Arduino firmware every time you make a change, and improves development efficiency.
> 
![](/5.Docs/Images/Holo4.jpg)

`ImageToHolo`文件夹下包含一个Python脚本，用于将图片转换成HoloCubic固件中用到的图像资源。

The `ImageToHolo` folder contains a Python script for converting images to image resources used in HoloCubic firmware.


> 因为图像资源一般都比较占空间，如果全部存在ESP32的Flash中的话存不了几张，因此我在框架中移植了LVGL的FAT文件系统支持，可以将图片资源存储在SD卡内进行读取。
> 
> Because image resources generally take up more space, if all exist in the ESP32 flash can not save a few, so I ported LVGL's FAT file system support in the framework, you can store the image resources in the SD card for reading.
> 
> 官方的图转换工具是在线的：[https://lvgl.io/tools/imageconverter](https://lvgl.io/tools/imageconverter) ，需要选择 `Indexed 4 colors` 格式。
> 
>The official chart conversion tool is online at [https://lvgl.io/tools/imageconverter](https://lvgl.io/tools/imageconverter) and requires the selection of the `Indexed 4 colors` format.
> 


HoloCubic用到的图片资源名为`xxx.bin`文件，用提供的脚本转好后放入SD卡，像这样读取：

HoloCubic uses an image resource called the `xxx.bin` file, which is spun up with the provided script and placed on the SD card and read like this:

```
lv_obj_t* imgbtn = lv_imgbtn_create(lv_scr_act(), NULL);
lv_imgbtn_set_src(imgbtn, LV_BTN_STATE_PRESSED, "S:/dir/icon_pressed.bin");
lv_imgbtn_set_src(imgbtn, LV_BTN_STATE_RELEASED, "S:/dir/icon_released.bin");
```

其中`S:`指代SD卡根目录（注意**S是大写的**），后面就是跟Linux中的路径完全表示一致了。

Where `S:` refers to the root directory of the SD card (note that **S is capitalized **), followed by a path that is exactly the same as in Linux.





**另外由于转换脚本的使用需要再Python环境下，如果不想安装环境的话，也可以用预编译好的exe文件来转，把`jpg/png/bmp`图片拖到`holo转换器.exe`的图标上（可以同时拖动多个上去），会在当前目录生成对应的`.bin`文件。**

**Additionally, since the conversion script requires Python environment, if you don't want to install the environment, you can also use the pre-compiled exe file to convert, drag the `jpg/png/bmp` images to the `holo converter.exe` icon (you can drag more than one up at the same time), it will generate the corresponding `.bin` file in the current directory. **

> 转换器软件的下载地址：
> 
> Download address for the converter software:
>
> 链接：https://pan.baidu.com/s/11cPOVYnKkxmd88o-Ouwb5g  提取码：xlju 




