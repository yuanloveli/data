### 虚拟机CentOS 7 如何连接网络

**step1**:在VMware界面单机虚拟机，进入初始界面后，首先点击  **虚拟机** **->设置->网络适配器->选择NAT模式**.

step2:打开CenOS 7终端 在命令行中输入**ifconfig回车**，查看虚拟机的网络配置，注意第一行显示为**ens32**

step3:在命令行中输入su root,登录完成后接着输入**vi /etc/sysconfig/network-scripts/ifcfg-ens32回车**，（此处命令可能有略微变化，如果上一步中输入ifconfig回车后显示的为ens33或其他，则命令相应改成vi /etc/sysconfig/network-scripts/ifcfg-ens33回车或其他），出现网络配置相关选项，此时可以看到ONBOOT选项值为no，输入i进入编辑模式，改为yes，按Esc输入:wq保存退出。

step4:输入**ping www.baidu.com回车**,得到反馈则网络配置成功。