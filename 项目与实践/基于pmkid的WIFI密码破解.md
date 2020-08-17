打比赛时遇到的一道有意思的题目，角度挺新颖，让我吃了固定思维的亏。

题目是一个压缩包，里面有一个`pcapng`文件，压缩包注释还贴心地给了密码的格式，`DASCTF`加4个字母：

![image-20200817233058463](https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/pmki/image-20200817233058463.png)

在`wireshark`中查看数据包内容，发现了`wifi`握手包。

![image-20200817233357162](https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/pmki/image-20200817233357162.png)

结合压缩包的注释，猜想应该是密码爆破没跑了，果断`aircrack-ng`走起：

```shell
crunch 10 10  -t DASCTF%%%% > a.txt # 生成DASCTF+4位数密码字典
aircrack-ng  -w a.txt dasctf.pcap #爆破密码 
```

结果没找到密钥，不按套路出牌：

![image-20200817233650836](https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/pmki/image-20200817233650836.png)

用其他的生成字典试了下，比如`/DASCTFxxxx/`、大小写变换等，都不行，我甚至开始怀疑压缩包注释是不是在搞事。

后续上网冲浪了下，发现`WPA`密码爆破的方法不止一个，有个只利用`PMKID`的方法，刚好在`aircrack-ng`的输出中存在这个信息，可以尝试一下。首先安装组件，其中`hcxdumptool`为数据包嗅探组件，`hcxtools`是数据格式转换组件，`hashcat`是最终利用的爆破组件：

```shell
apt-get install libcurl4-openssl-dev libssl-dev zlib1g-dev libpcap-dev
git clone https://github.com/ZerBea/hcxtools
cd hcxtools
make
make install

git clone https://github.com/ZerBea/hcxdumptool
cd hcxdumptool
make
make install

wget https://github.com/hashcat/hashcat/releases/download/v6.1.1/hashcat-6.1.1.7z
7z x hashcat-4.2.1.7z
```

配置好组件后，使用`hcxpcaptool`将题目提供的`pcapng`文件转换为`hashcat`可使用的格式：

```shell
hcxpcaptool -z pmkid.16800 /home/kali/Desktop/dasctf.pcapng
```

![image-20200817234511767](https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/pmki/image-20200817234511767.png)

工具显示了数据包的信息，同时将`pmkid`写入了`pmkid.16800`,每一行由星号分成4块，分别是：PMKID、AP的MAC地址、客户端的MAC地址、ESSID。

```shell
kali@kali:~/Desktop$ cat pmkid.16800                                                                                             8552c996486d61fce8a70fc8b78532a3*02bd18082865*ac88fd01554f*444153435446
```

最后用`hashcat`加载压缩包所提供的信息字典进行爆破，解出密码`DASCTF5598`：

```shell
./hashcat.exe -a 0 -w 3 -m 16800 C:\Users\hjc\Desktop\pmkid.16800 C:\Users\hjc\Desktop\a.txt
```

![image-20200818000301826](https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/pmki/image-20200818000301826.png)