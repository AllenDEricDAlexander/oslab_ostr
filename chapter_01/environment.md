## 环境准备

```
vmware + deepin-desktop-community-20.8-amd64

（已在纯净ubuntu 22.04上验证过此教程，完全可行！只是需要安装vim 命令： sudo apt install vim             -- by 看山）
```

```shell
# 环境准备
sudo apt install build-essential

sudo apt-get install libghc-x11-dev

sudo apt-get install xorg-dev

sudo apt install nasm

# 到你想去的目录 我的是 /home/allen/oslib

wget https://udomain.dl.sourceforge.net/project/bochs/bochs/2.6.8/bochs-2.6.8.tar.gz

tar -zxvf bochs-2.6.8.tar.gz

mkdir bochs

cd bochs-2.6.8

./configure --prefix=/home/allen/oslib/bochs --enable-debugger --enable-disasm --enable-iodebug --enable-x86-debugger --with-x --with-x11 LDFLAGS='-pthread'

make

make install

cd ..

cd bochs

touch bochsrc.disk

vim bochsrc.disk

# 写入以下内容  注意路径！！！此后不再提示
---
megs : 32

romimage: file=/home/allen/oslib/bochs/share/bochs/BIOS-bochs-latest
vgaromimage: file=/home/allen/oslib/bochs/share/bochs/VGABIOS-lgpl-latest

boot: disk

log: bochs.out

mouse:enabled=0
keyboard:keymap=/home/allen/oslib/bochs/share/bochs/keymaps/x11-pc-us.map

ata0:enabled=1,ioaddr1=0x1f0,ioaddr2=0x3f0,irq=14
ata0-master: type=disk, path="hd60M.img", mode=flat,cylinders=121,heads=16,spt=63

#gdbstub:enabled=1,port=1234,text_base=0,data_base=0,bss_base=0


---

# 启动磁盘
bin/bximage

# 然后在输入框依次输入以下，输入一个，按一次回车

1

hd

flat

60

hd60M.img
```

## 测试

```shell
#  测试代码

cd ..

touch mbr.s

vim mbr.s

---
SECTION MBR vstart=0x7c00
	mov ax,0x0000	;设置栈指应该是程序一开始就应该做的事情,这个值是参照1m内存空间布局图选择的，以后会刻意避开
	mov ss,ax
	mov ax,0x7c00
	mov sp,ax	
 
	mov ax,0x0600
	mov bx,0x0700	;BH是设置缺省属性，属性是指背景色，前景色，是否闪烁等，例如07H表示黑底白字，70H表示灰底黑字等等。
	mov cx,0x0000
	mov dx,0x184f	;这个看书p61，同时看其中关于页的知识
	int 0x10
	
	mov ax,0x0300	
	mov bx,0x0000	
	int 0x10
	
	mov ax,0x0000
	mov es,ax
	mov ax,message
	mov bp,ax
	mov ax,0x1301
	mov bx,0x0007	;设置字体属性，02是黑底绿字，07是黑底白字
	mov cx,0x000c
	int 0x10
	
	jmp $
	message db "Hello World!"
	times 510-($-$$) db 0
	db 0x55,0xaa
---

# 编译
nasm -o test mbr.s

# 写入虚拟机启动盘
dd if=/home/kanshan/Desktop/test of=/home/kanshan/Desktop/bochs/hd60M.img bs=512 count=1 conv=notrunc

# 启动查看效果
cd bochs

bin/bochs -f bochsrc.disk

# 输入c 即可看到 hello world
```

