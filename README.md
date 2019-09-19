
里面有些io地址是错误的，还需修改验证



<-------------------------------------------------蓝牙打不开-------------------------------------------->

例rk3288

GRF base : 0xff77000

GPIO4_C0 ~ GPIO4_C3

GPIO4 base 0xff7B0000

A检查 iomux 是否正常

io -4  0xff770044

最低字节为 0x55 说明 iomux 正常

【注】 B C 部分的测试 请先执行 echo 1 > /sys/class/rfkill/rfkill0/state 确保先把蓝牙的供电打开

B 使用附件工具关闭蓝牙 bt_dbg 测试 主控的TX 端是否有方波，如果无法测量到 方波 请做 C部分的 导通性测试

C 导通性测试

1 配置为 gpio

io -4  -w 0xff770044 0xffff0000

2 配置为 output

io -4 -w 0xff7b0004 0x00ff000f

3 输出高电平 测量 RX TX CTS RTS 是否为高电平

io -4 -w 0xff7b0000 0x00ff0000

输出低电平 测量 RX TX CTS RTS 是否为低电平

io -4 -w 0xff7b0000 0x00000000

如果 导通性 测试不过 说明 硬件 存在问题 请检查哪个地方 是否有短路 等

GPIO4_C0 ~ GPIO4_C3

1 配置为 gpio

io -4 0xff770044 0xffff0000

2 配置为 output

io -4 -w 0xff7b0004 0x00ff000f

3 输出高电平 测量 RX TX CTS RTS 是否为高电平

io -4 -w 0xff7b0000 0x00ff0000

输出低电平 测量 RX TX CTS RTS 是否为低电平

io -4 -w 0xff7b0000 0x00000000

pin 136 (gpio4-16): ff180000.serial (GPIO UNCLAIMED) function uart0 group uart0-xfer

pin 137 (gpio4-17): ff180000.serial (GPIO UNCLAIMED) function uart0 group uart0-xfer

pin 138 (gpio4-18): ff180000.serial (GPIO UNCLAIMED) function uart0 group uart0-cts

pin 139 (gpio4-19): wireless-bluetooth gpio4:1139 function uart0 group uart0-rts

echo 136 > /sys/class/gpio/export

echo out > /sys/class/gpio/gpio136/direction

echo 137 > /sys/class/gpio/export

echo out > /sys/class/gpio/gpio137/direction

echo 138 > /sys/class/gpio/export

echo out > /sys/class/gpio/gpio138/direction

echo 139 > /sys/class/gpio/export

echo out > /sys/class/gpio/gpio139/direction

echo 1 > /sys/class/gpio/gpio136/value

echo 1 > /sys/class/gpio/gpio137/value

echo 1 > /sys/class/gpio/gpio138/value

echo 1 > /sys/class/gpio/gpio139/value

cat /sys/kernel/debug/gpio

echo 0 > /sys/class/gpio/gpio136/value

echo 0 > /sys/class/gpio/gpio137/value

echo 0 > /sys/class/gpio/gpio138/value

可能串口导通性有问题。

先确认IOMUX关系，312x 需要禁用内部上拉，外部接电阻上拉。参考附件去修改。

还有UART是交叉接的，确认是否有问题。请把结果一次性做完整，上传所有结果。

正基的模组是需要接外部晶振的，需要确认是否起振或者频偏是多少，是否是达到+-10ppm 越小越好啦，还峰峰值也需要确认


serial 20060000.serial: error:lsr=0xd9

这一般是UART TX, RX 数据线没有维持在高电平导致异常

Rk312x 平台，可以通过以下修改禁掉UART0 口内部的下拉，

类似如下：

uart0_xfer: uart0-xfer {

rockchip,pins = <UART0_SIN>,

<UART0_SOUT>;

rockchip,pull = <VALUE_PULL_UPDOWN_DISABLE>;

};

<-------------------------------------------------wifi打不开-------------------------------------------->

开机启动流程:

wifi rfkill -mmc扫卡-> dhd mmc rescan 扫卡（mmc sdio）

[WifiHW  : uevent path:/sys/bus/sdio/devices/./uevent

WifiHW  : uevent path:/sys/bus/sdio/devices/../uevent

WifiHW  : uevent path:/sys/bus/sdio/devices/mmc2:0001:1/uevent]  -> 加载patch -> wlan0[/proc/net/dev]

-> wpa_supplicant(dhd和rfkill一起玩耍，起后台)-> IPC connect supplicant



driver/wireless/rockchip_wlan/rkwifi/bcmdhd/dhd_sdio.c


WL_REG_ON电平测

Echo 0 > /sys/class/rkwifi/power

Echo 1 > /sys/class/rkwifi/power



andorid8.1之前选择性wifi驱动在wifi service 启动之后再运行

CONFIG_WIFI_LOAD_DRIVER_WHEN_KERNEL_BOOTUP=n

echo 1 > /sys/class/rkwifi/driver


外部晶体频偏，频偏比较大情况下，会出现能扫描到模块但是初始化失败，峰峰值>=0.7*VDDIO && 峰峰值 <= 1*VDDIO，频偏过大峰峰值错误会影响wifi（扫描连接热点）和蓝牙（扫描连接设备））

降低sdio_clk ，降低clk可以，考虑硬件上走线；如果降低clk依然不行，考虑使用sdio单线模式,如果单线模式可以而使用4线模式不行，检查硬件上sdio_data0~sdio_data3 四根线的线序是否弄错


电源这块一共就是3个脚，正基是分开供电

VCC_WLIN

WL_REG_ON

BT_RST


WIFI部分看VCC_WLIN 还有WL_REG_ON就可以了，AP6xxx 系列模块 模组外部供晶振32.768K check是否正常




RK312x 平台，SDIO 接口需要接外部上拉电阻，并将默认芯片的上下拉禁掉

sdio0_cmd: sdio0_cmd {

rockchip,pins = <MMC1_CMD>;

'-' rockchip,pull = <VALUE_PULL_DISABLE>;

'+' rockchip,pull = <VALUE_PULL_UPDOWN_DISABLE>;

};


RK3288 ，主控的SDIO 电平有1.8、3.3V 可配置，需要根据实际硬

件(VCCIO_WL 模组的电源电压 ）来配置dts 中的：sdio_vref = <1800>;可以先量一下







