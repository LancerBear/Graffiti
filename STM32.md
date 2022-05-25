# 名词解释

## STLink

- SWCLK -- SWCLK / JTCK
- SWDIO -- SWDIO / JTMS
- 3.3V -- VCC3.3
- GND -- GND

## JTAG （Joint Test Action Group，联合测试行动小组）

- TMS：测试模式选择，TMS用来设置JTAG接口处于某种特定的测试模式；
- TCK：测试时钟输入；
- TDI：测试数据输入，数据通过TDI引脚输入JTAG接口；
- TDO：测试数据输出，数据通过TDO引 脚从JTAG接口输出；

## SWD （Serial Wire Debug，串行调试）

- 相比于JTAG，只需要4个引脚
- 高速模式更可靠，大数据量JTAG可能会下载程序失败

## OpenOCD (Open On-Chip Debugger)

- 开源的片上调试器
- 需要配合调试仿真器（stlink 、jlink）
- 集成gdb server，因此可以实现gdb在片上的调试
- 集成telnet server，因此可以对目标板进行程序烧录

# Linux下环境搭建

## gcc-arm-none-eabi -- 编译工具链

- 下载 sudo apt install gcc-arm-none-eabi
- https://blog.csdn.net/ybhuangfugui/article/details/98136988
- 使用arm-none-eabi-gcc工具来进行编译，类比于PC环境下的gcc



## stm32标准外设库

- 下载地址https://www.st.com/zh/embedded-software/stm32-standard-peripheral-libraries.html，选取对应系列芯片，此处选F1

- 下载解压出STM32F10x_StdPeriph_Lib_V3.6.0文件夹，重点关注Libraries文件夹

  ```shell
  Lancer@Lancer-PC:~/Documents/Codes/stm32/STM32F10x_StdPeriph_Lib_V3.6.0/Libraries$ tree  -L 2
  .
  ├── CMSIS
  │   ├── CM3 # 启动相关文件，包括内核文件、初始化、启动的汇编文件
  │   ├── CMSIS changes.htm
  │   ├── CMSIS debug support.htm
  │   ├── Documentation
  │   └── License.doc
  └── STM32F10x_StdPeriph_Driver # 官方提供的各种外设API，一般直接把src inc目录所有文件拷贝到自己的工程目录下
      ├── inc
      ├── LICENSE.txt
      ├── Release_Notes.html
      └── src
  
  6 directories, 5 files
  
  ```

  ```shell
  Lancer@Lancer-PC:~/Documents/Codes/stm32/STM32F10x_StdPeriph_Lib_V3.6.0/Libraries/CMSIS/CM3$ tree  
  .
  ├── CoreSupport # 内核文件，需要拷贝到自己创建的工程文件夹中
  │   ├── core_cm3.c
  │   └── core_cm3.h
  └── DeviceSupport
      └── ST
          └── STM32F10x
              ├── LICENSE.txt
              ├── Release_Notes.html
              ├── startup # 启动文件，区分不同环境
              │   ├── arm
              │   ├── gcc_ride7
              │   ├── iar
              │   └── TrueSTUDIO # TrueSTUDIO使用的gcc编译，和当前使用的环境一致，因此需要使用该文件夹下的启动文件
              │       ├── startup_stm32f10x_cl.s
              │       ├── startup_stm32f10x_hd.s # 当前使用的STM32F103ZET6 512K使用这个启动文件
              │       ├── startup_stm32f10x_hd_vl.s
              │       ├── startup_stm32f10x_ld.s
              │       ├── startup_stm32f10x_ld_vl.s
              │       ├── startup_stm32f10x_md.s
              │       ├── startup_stm32f10x_md_vl.s
              │       └── startup_stm32f10x_xl.s
              ├── stm32f10x.h # 总和头文件，需要拷贝到自己工程目录
              ├── system_stm32f10x.c # 系统相关文件，SystemInit函数在其中，需要拷贝到自己工程目录
              └── system_stm32f10x.h
  
  9 directories, 39 files
  
  ```

  

## 工程目录组织

├── Libraries
│   ├── inc
│   └── src
├── Start
│   ├── core_cm3.c
│   ├── core_cm3.h
│   ├── startup_stm32f10x_hd.s
│   ├── stm32f10x.h
│   ├── system_stm32f10x.c
│   └── system_stm32f10x.h
├── User
│   ├── main.c
│   ├── stm32f10x_conf.h
│   ├── stm32f10x_it.c
│   └── stm32f10x_it.h
└── makefile

5 directories, 11 files

## OpenOCD -- 烧写工具

- 用于烧录程序

- sudo apt-get install openocd

- 安装后脚本目录在/usr/share/openocd/scripts

- 使用方法 https://cloud.tencent.com/developer/article/1662697

  ```shell
  # 1、插入stlink，使用openocd连接
  Lancer@Lancer-PC:~$ openocd -f /usr/share/openocd/scripts/interface/stlink-v2.cfg -f /usr/share/openocd/scripts/target/stm32f1x.cfg  
  Open On-Chip Debugger 0.10.0
  Licensed under GNU GPL v2
  For bug reports, read
          http://openocd.org/doc/doxygen/bugs.html
  Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
  Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
  adapter speed: 1000 kHz
  adapter_nsrst_delay: 100
  none separate
  Info : Unable to match requested speed 1000 kHz, using 950 kHz
  Info : Unable to match requested speed 1000 kHz, using 950 kHz
  Info : clock speed 950 kHz
  Info : STLINK v2 JTAG v29 API v2 SWIM v7 VID 0x0483 PID 0x3748
  Info : using stlink api v2
  Info : Target voltage: 3.304043
  Info : stm32f1x.cpu: hardware has 6 breakpoints, 4 watchpoints
  Info : accepting 'telnet' connection on tcp/4444
  
  
  # 2、另外开启一个terminal, 使用telnet 登录到openocd开放的4444端口
  Lancer@Lancer-PC:~$ telnet localhost 4444
  Trying ::1...
  Trying 127.0.0.1...
  Connected to localhost.
  Escape character is '^]'.
  Open On-Chip Debugger
  
  # 3、halt命令，停止目标机运行
  > halt 
  target halted due to debug-request, current mode: Handler HardFault
  xPSR: 0x01000003 pc: 0xfffffffe msp: 0xffffffdc
  
  # 4、falsh write_image erase xxx把xx文件下载到目标机中
  > flash write_image erase /home/Lancer/Template.hex 
  auto erase enabled
  device id = 0x10016414
  flash size = 512kbytes
  target halted due to breakpoint, current mode: Handler HardFault
  xPSR: 0x61000003 pc: 0x2000003a msp: 0xffffffdc
  wrote 2048 bytes from file /home/Lancer/Template.hex in 0.155276s (12.880 KiB/s)
  
  # 5、reset 复位目标机，复位之后下载的程序即会运行
  > reset
  
  # 6、ctrl + ] 退出telnet终端
  ```
  
  ​	
  
  ```shell
  # 也可以直接使用openocd -c 参数下发命令，一键下载
  openocd -f /usr/share/openocd/scripts/interface/stlink-v2.cfg -f /usr/share/openocd/scripts/target/stm32f1x.cfg -c init -c halt -c "flash write_image erase /home/Lancer/Template.hex" -c reset -c shutdown
  # 注意flash write_image erase带引号
  # 可以考虑将上述命令写入makefile，可以直接make download一键下载
  ```