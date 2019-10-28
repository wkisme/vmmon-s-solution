# vmmon-s-solution
solution for can't load vmmon, which is caused by signature authentication.

***
This article is reprint from https://blog.csdn.net/jinking01/article/details/82763968.
***
I want to put here to remind me how to solve this problem when it appears again.

版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
本文链接：https://blog.csdn.net/jinking01/article/details/82763968

今天同事在ubuntu16.04下安装vmware后,想安装win10的系统,结果报错:Cannot open /dev/vmmon: No such file or directory,Please make sure that the kernel module `vmmon' is loaded这个错误大概是说vmmonitor和vmnet这俩模块没有经过签名认证,可能是不安全的,所以在安装vmware的时候,出于安全的考虑,无法built.

从网上搜到的一些解决方案,大部分都是说要启动bios后修改scure boot,将之修改为disable,同事照做之后,无法进入ubuntu.

后面我搜索了一下,发现在不修改bios的情况下,也有可能修复这个错误.(网址:https://kb.vmware.com/s/article/2146460,但是这个网页里面有一些是错误的).现在把我们修复的过程贴出来:

1. 使用openssl生成rsa2048公钥对,将来用于对 vmmon 和 vmnet模块进行签名:

$openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=VMware/"

这里MOK 是你生成的公钥对的名字,可以修改,会生成俩文件,MOK.der和MOK.priv,发布者为"/CN=VMware/"

2. 签名:

$sudo /usr/src/linux-headers-`uname -r`/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmmon)

$sudo /usr/src/linux-headers-`uname -r`/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmnet)

或者

$sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmmon)

$sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmnet)

3. 将公钥MOK.der导入到MOK列表:

$sudo mokutil --import MOK.der

然后输入两次密码,记住这个密码(在重启后的mok enrollemnt里面还会用到)

4. 重启

等待一段时间,进入MOK management控制台,选择"mok enrollment"选项,然后yes,然后reboot,等待一段时间后,进入ubuntu页面,启动vmware,一切正常.

如果还是不行,则对kernel下的vmmon和vmnet签名:

$ sudo /usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmmon)

$ sudo /usr/src/kernels/$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmnet)
————————————————
版权声明：本文为CSDN博主「土豆西瓜大芝麻」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/jinking01/article/details/82763968
