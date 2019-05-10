

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


<-------------------------------------------------wifi打不开-------------------------------------------->

