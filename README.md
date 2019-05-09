先搞一下io mux

这种就是完全要分析datasheet的

Rockchip RK3288TRM V1.2 Part1-20170321.pdf



在TRM上面是有的，可以看到




我现在要用的就是GRF和GPIO4

用这两个就可以配置蓝牙的tx rx cts rts bt_en 


首先看一下GRF base : 0xff77000
怎么去设置GPIO4的复用


硬件上接的是GPIO4_C0 ~ GPIO4_C3
对应rx tx cts rts 



base  ff770000

找到了offset就是0x0044，启动设置的值就是0x00000000 一共4个字节 一共32位

4c0 4c1 4c2 4c3 4c4 4c5 4c6 4c7  8个引脚

一个引脚4位

1 配置为 gpio
 	io -4 --w 0xff770044 0xffff0000


配置c0 c1 c2 c3 配置的是高位 反正从左到右

0x 0   0  0   0 0 0 0 0 
     c0 c1 c2 c3 


找到了就有的说的





       01010101
如果要设置成为串口的话，就是要设置低八位为0x55，不如稀里糊涂设置一个0x55确实不舒服



31：16 是write enable 如果没有设置成为高的意思就是写不了喔

那可以了

0x5555

io -4  0xff770044

正常读出来应该是0x00005555，如果都复用了的话。。是不是

如果要test的时候，就将设置为0xffff0000
然后rx tx cts rts sdio0~3  都可以测试了


嗯呢，这个是复用的


然后是引脚设置，这个也看看



配置为 output
 	io -4 -w 0xff7b0004 0x00ff000f
 	3 输出高电平 测量 RX TX CTS RTS 是否为高电平
 	io -4 -w 0xff7b0000 0x00ff0000
 	输出低电平 测量 RX TX CTS RTS 是否为低电平
 	io -4 -w 0xff7b0000 0x00000000
 	如果 导通性 测试不过 说明 硬件 存在问题 请检查哪个地方 是否有短路 等


然后再看看offset 0xff7b0004


这个MAP是配置GPIO4的，



gpio4 
base ---- ff7b0000

一共有32个脚
一共有32位

io -4 -w 0xff7b0004 0x00ff000f

这个是啥

GPIO4   C0 C1 D2 C3 
               16  17 18  19 


0x00ff000f   

不是一个f就行了吗？？看来是其它引脚也设置了。。。0是input 1 是output 一位就是对应一个引脚

没毛病。。。。。


io -4 -w 0xff7b0000 0x00ff0000
 后面这个就没毛病了。。。
io -4 -w 0xff7b0000 0x00000000

这样过一遍对其它ic也是大同小异的方法，会舒服很多



OK  那就可以看一下方法了


1、首先io -4 0xff770044，看看iomux 低八位是否是0x55

2、echo 1 > /sys/class/rfkill/rfkill0/state 蓝牙供电

3、io导通性测试
     io -4 -w 0xff770044 0xffff0000
     io -4 -w 0xff7b0004 0x00ff000f
     io -4 -w 0xff7b0000 0x00ff0000
     io -4 -w 0xff7b0000 0x00000000
