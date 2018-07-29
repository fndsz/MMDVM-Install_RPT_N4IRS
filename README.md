# MMDVM-Install

Scripts and config files to run the MMDVM on Debian

Start MMDVMHost at boot time (systemd)

Restart MMDVM on failure (systemd) 

Log to /var/log

provide a method to upload new firmware to the MMDVM (BOSSA)

provide a method to download, compile and install a new version of MMDVMHost when available

N4IRS

# BY BI7JTA 编译过程   

需要:stm32flash   
https://sourceforge.net/p/stm32flash/wiki/Home/   

pi-star@pi-star(rw):MMDVM$ make -f Makefile.CMSIS    
   
mkdir -p obj   
   
mkdir -p bin   

arm-none-eabi-g++ -MMD -mthumb -mlittle-endian -mcpu=cortex-m3 -Wall -I. -I/opt/STM32Cube_FW_F1_V1.4.0/Drivers/CMSIS/Include -I/opt/STM32Cube_FW_F1_V1.4.0/Drivers/CMSIS/Device/ST/STM32F1xx/Include -Isystem_stm32f1xx  -DSTM32F105xC -Os -flto -ffunction-sections -fdata-sections -g  -nostdlib -fno-exceptions -fno-rtti -c IOSTM.cpp -o obj/IOSTM.o   
   
make: arm-none-eabi-g++: Command not found   
   
Makefile.CMSIS:129: recipe for target 'obj/IOSTM.o' failed   
   
Solution:   
sudo apt-get install gcc-arm-none-eabi   
sudo apt-get install build-essential   
   
Catch error,    
/usr/bin/mandb: can't create index cache /var/cache/man/18547: No space left on device   
Errors were encountered while processing:
 /var/cache/apt/archives/libstdc++-arm-none-eabi-newlib_4.8.3-9+4_all.deb
E: Sub-process /usr/bin/dpkg returned an error code (1)   
   
Solution：   
sudo pistar-expand   
Ignore any error and reboot pi-star OS   
   
Run again,   
sudo apt-get install gcc-arm-none-eabi   
sudo apt-get install build-essential   
   
sudo make -f Makefile.CMSIS program   
Globals.h:27:23: fatal error: stm32f1xx.h: No such file or directory   
 #include "stm32f1xx.h"   
                          ^   
compilation terminated.      
Makefile.CMSIS:129: recipe for target 'obj/IOSTM.o' failed   
make: *** [obj/IOSTM.o] Error 1   
   
相关编译：   
https://github.com/wojciechk8/MMDVM_pog   
https://github.com/wojciechk8/MMDVM_pog/issues/2   
https://github.com/N4IRS/MMDVM-Install/tree/master/STM32-DVM   
   
德国DF2ET https://www.df2et.de/mmdvm_hs_builder/   
   
重新开始：   
1，参照指导 https://github.com/N4IRS/MMDVM-Install/tree/master/STM32-DVM   
原始 https://github.com/wojciechk8/MMDVM_pog   
直接下载固件：   
http://dvswitch.org/files/HAM/MMDVM/   
	STM32DVM_NXDN.zip	2018-02-18 07:21	57K	    
[DIR]	Version_2_Firmware/	2018-07-15 19:39	-	    
[DIR]	Version_3_Firmware/	2018-05-12 20:23	-	    
[   ]	mmdvm.hex	2018-01-17 20:44	132K	    
[   ]	mmdvm_cos.hex	2018-01-17 20:44	133K	    
[   ]	mmdvm_nxdn.hex	2018-02-27 05:58	140K	    
[   ]	p25-dashboard.tar	2018-01-06 06:25	60K	    
[   ]	stm32cube_fw_f1_v140.zip	2017-06-14 14:29	77M	    
[   ]	stm32flash   
   
2,准备 STM32Cube_FW_F1_V1.4.0 编译环境，   
下载 https://github.com/cody82/stm32-rust/find/master   
解压，将目录STM32Cube_FW_F1_V1.4.0 移动到 /opt/   
   
3，刷新固件，   
cd /usr/src/MMDVM   
3.1 GPIO方式   
For the STM32-DVM PiHat    
Shutdown the Raspberry Pi        树莓派关机   
Disconnect power to Raspberry Pi 断电   
Insert JP1 jumper                插入JP1升级跳线,板上的Boot   
Power ON the Raspberry Pi        上电   
PWR, ACT and DMR should be lit solid, NOT flashing.    
                                 PWR ACT DMR 微亮，不闪烁   
   
make -f Makefile.CMSIS    
program-STM32_DVM_PiHat   
Shutdown the Raspberry Pi   
Disconnect power to Raspberry Pi   
remove JP1 jumper   
Power ON the Raspberry Pi   
start MMDVMHost   
   
#   
rpi-rw   
源码目录：cd /usr/src/MMDVM   
cd /srv/MMDVM-Install/STM32-DVM   
串口：   
sudo stm32flash -w /home/pi-star/mmdvm_F105_Ver0722.hex -v /dev/ttyAMA0   
USB：   
sudo stm32flash -w /home/pi-star/mmdvm.hex -v /dev/ttyAMA0   
sudo stm32flash -w /home/pi-star/mmdvm.hex -v /dev/ttyUSB0   
GIHUB版本   
sudo stm32flash -w /home/pi-star/mmdvm_F105_19.2_Ver180223.hex -v /dev/ttyAMA0   
sudo stm32flash -w /home/pi-star/mmdvm_F105_19.2_Ver180223.hex -v /dev/ttyUSB0   
   
重新编译的版本：   
sudo stm32flash -w /usr/src/MMDVM/bin/mmdvm.hex -v /dev/ttyAMA0   
sudo stm32flash -w /usr/src/MMDVM/bin/mmdvm.hex -v /dev/ttyUSB0   
      
停止服务进入boot模式   
sudo systemctl stop pistar-watchdog   
sudo systemctl stop mmdvmhost   
sudo mv /etc/mmdvmhost /etc/mmdvmhost.save   
   
启动服务   
sudo mv /etc/mmdvmhost.save /etc/mmdvmhost   
sudo systemctl start pistar-watchdog   
#插上GPIO/USB HAT   
sudo systemctl start mmdvmhost   
   
修改Config.h   
// For 19.2 MHz   
 #define EXTERNAL_OSC 19200000   
   
编译：   
sudo make -f Makefile.CMSIS clean   
sudo make -f Makefile.CMSIS program   
   
无法进入刷机模式的解决方法   
https://github.com/N4IRS/MMDVM-Install/issues/8   
   
可能的问题：   
1，GPIO，成功率低，STM32烧毁的危险   
2，PiZERO电压不够，GPIO热插断电，   
3，USB模式，已经入刷机模式，写固件时， ttyUSB0不可识别，   
  检查：pi-star@pi-star(ro):~$ ls /dev/ttyU   
  改为：ttyUSB1     
  USB模式，无法进入刷机模式，   
   
4，GPIO模式，无法进入蓝灯模式，   
mv /etc/mmdvmhost /etc/mmdvmhost.save   
断电重启，待启动完成后，在短接BootLoader，插入GPIO   
5，写入完成后HAT上电无法自检，固件问题   
6，可以自检，但ACT灯不良，检查ttyUSB1，默认为ttyUSB0,需要重启   
   
检查是否成功，   
   
   
      
快速进入蓝灯（DMR/LED/PWD)模式   
Try renaming /etc/mmdvmhost.   
login   
gain root access   
rpi-rw   
cd /etc   
sudo mv /etc/mmdvmhost /etc/mmdvmhost.save   
poweroff   
Put board in BL mode.   
      
==========================   
最新代码编译失败， N4IRS 提示用 6月6号的MMDVM源码版本版本，并覆盖SerialRB.h    
https://github.com/N4IRS/MMDVM-Install/issues/8   
“   
，Revert back to this commit   
，<https://github.com/g4klx/MMDVM/tree/d6c1bea80ae1fd150111bb7a692bb5320f3beed0>   
，Edit the config to match your board and make:   
，make -f Makefile.CMSIS   
，Make sure you replace the SerialRB.h   
”   
https://github.com/g4klx/MMDVM/tree/d6c1bea80ae1fd150111bb7a692bb5320f3beed0   
   
编译后可以boot，但存在问题：   
1，DMR灯不亮，   
2，RX解码困难误码率高，   
3, TX无法解码   
   
解决：执行 clean   
sudo make -f Makefile.CMSIS clean   
sudo make -f Makefile.CMSIS   

编译结果：  
http://mmdvm.io/files/STM32DVM_F105/  

## Flash use USB or GPIO 
Enter flash mode  
1）Use USB mode  
1.1 rpi-rw #make system writeable  
1.2 sudo mv /etc/mmdvmhost /etc/mmdvmhost.save #make service down  
  
1.3 Disconnect STM32-DVM from the RPi host, GPIO and USB all disconnect,  
1.4 Insert JP jumper, short BOOT and VCC near P25 LED,  
1.5 Connect STM32-DVM to the RPi host use USB wire,  
1.6 Endter flash mode. PWR, ACT and DMR should be lit solid, NOT flashing(Very important,if not disconnect USB and try again) .  
1.7 Flash use USB (replace your .hex path, and download stm32flash,see Require libs )  
sudo stm32flash -w /usr/src/MMDVM/bin/mmdvm.hex -v /dev/ttyUSB0  
  
Require libs  
https://github.com/N4IRS/MMDVM-Install/blob/master/STM32-DVM/required-libs.sh  

## Flash use ST-LINK


