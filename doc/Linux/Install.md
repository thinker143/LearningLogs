# 创建虚拟机

1、在主页点击创建新的虚拟机

<img src="C:\Users\windows10\AppData\Roaming\Typora\typora-user-images\image-20241015121445213.png" alt="image-20241015121445213" style="zoom:33%;" />

2、点击下一步

<img src="C:\Users\windows10\Desktop\36cb8ad5d442f0f0cb539cf9d673587.png" alt="36cb8ad5d442f0f0cb539cf9d673587" style="zoom: 33%;" />

3、选择准备好的IOS文件，点击下一步

<img src="C:\Users\windows10\AppData\Roaming\Typora\typora-user-images\image-20241015121102572.png" alt="image-20241015121102572" style="zoom:33%;" />

4、填写虚拟机名称，点击下一步

<img src="C:\Users\windows10\AppData\Roaming\Typora\typora-user-images\image-20241015121244943.png" alt="image-20241015121244943" style="zoom:33%;" />

5、修改最大磁盘大小，这里建议设置的大一点，否则将来不够用调整很麻烦，然后选择将虚拟磁盘存储为单个文件

<img src="C:\Users\windows10\AppData\Roaming\Typora\typora-user-images\image-20241015121608864.png" alt="image-20241015121608864" style="zoom:33%;" />

6、下一步后，自定义硬件

<img src="C:\Users\windows10\AppData\Roaming\Typora\typora-user-images\image-20241015121654905.png" alt="image-20241015121654905" style="zoom:33%;" />

7、设置虚拟机硬件，建议cpu给到4核，内存给到8G

<img src="C:\Users\windows10\AppData\Roaming\Typora\typora-user-images\image-20241015121815314.png" alt="image-20241015121815314" style="zoom:33%;" />

8、最后点击完成，虚拟机就创建完毕了

<img src="C:\Users\windows10\AppData\Roaming\Typora\typora-user-images\image-20241015121908905.png" alt="image-20241015121908905" style="zoom:33%;" />

# 安装CentOS7

1、启动刚刚创建的虚拟机，开始安装CentOS7系统

2、启动后需要选择安装菜单，将鼠标移入黑窗口中后，将无法再使用鼠标，需要按上下键选择菜单。选中Install Centos 7 后按下回车

3、然后会提示我们按下enter键继续

4、过一会儿后，会进入语言选择菜单，这里可以使用鼠标选择。选择中文-简体中文，然后继续

5、接下来，会进入安装配置页面，鼠标向下滚动后，找到系统-安装位置配置，点击

6、选择刚刚添加的磁盘，并点击完成

7、然后回到配置页面，这次点击《网络和主机名》，

​     在网络页面做下面的几件事情：

1. 修改主机名为自己喜欢的主机名，不要出现中文和特殊字符，建议用localhost
2. 点击应用
3. 将网络连接打开
4. 点击配置，设置详细网络信息

8、最后回到配置界面后，点击开始安装，并设置密码。
