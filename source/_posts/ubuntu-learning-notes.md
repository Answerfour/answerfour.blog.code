---
title: Ubuntu Learning Notes
date: 2024-02-16
tags: 
  - Linux
categories: 
cover: /media/featureimages/1.jpg
description: 智能小车无法联网问题的解决
---

> 事情的起因是我按照教程运行搭载在JetsonNano上的ubuntu系统，发现系统无法连接wifi，于是查找资料试图解决。当我最后解决问题整理完这篇笔记之后我在脑海中构建了我的思考过程，发现解决这个问题其实也只需四到五步的推理，但是事实总是如此，为了这几步的解决步骤，我花了两个下午两个晚上，这么一想其实我的效率还可以更高点，下次我从早上开始学习思考，再遇到类似的未知领域的问题是不是一天就能想出来呢？我陷入了好奇之中。
>
> 


$$
\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}
$$

### 1 网络拓扑结构的理解

#### 1.1 输入`ifconfig`，查看网络信息

```shell
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.16  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::4320:3860:46:8803  prefixlen 64  scopeid 0x20<link>
        ether 48:b0:2d:5b:dc:56  txqueuelen 1000  (Ethernet)
        RX packets 18660  bytes 1886816 (1.8 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 20777  bytes 6823167 (6.8 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 150  base 0xe000  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 2505  bytes 180228 (180.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2505  bytes 180228 (180.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

rndis0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 6a:64:62:35:cd:69  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

usb0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 6a:64:62:35:cd:6b  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.9.1  netmask 255.255.255.0  broadcast 192.168.9.255
        inet6 fe80::1acc:18ff:fe9b:24a4  prefixlen 64  scopeid 0x20<link>
        ether 18:cc:18:9b:24:a4  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 1  overruns 0  frame 0
        TX packets 315  bytes 74205 (74.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### 1.2 理解命令行的基本内容

1. `lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536`

   - `lo`：网卡名称为 lo，代表本地回环接口。

   - `flags=73<UP,LOOPBACK,RUNNING>`：表示该接口卡的状态标志，其中：
     - `UP`：接口卡已启用。
     - `LOOPBACK`：接口卡为本地回环接口。
     - `RUNNING`：接口卡正在运行。
     
   - `mtu 65536`：接口卡的最大传输单元 (MTU) 大小为 65536 字节。
   
2. `inet 127.0.0.1  netmask 255.0.0.0`

   - `inet 127.0.0.1`：接口卡的 IPv4 地址为 127.0.0.1，它是本地回环地址，用于在本机内部进行通信。
   - `netmask 255.0.0.0`：子网掩码为 255.0.0.0，表示回环地址的范围。

3. `inet6 ::1  prefixlen 128  scopeid 0x10<host>`

   - `inet6 ::1`：接口卡的 IPv6 地址为 ::1，它是本地回环地址的 IPv6 版本。
   - `prefixlen 128`：IPv6 地址的前缀长度为 128。
   - `scopeid 0x10<host>`：接口卡的作用域 ID，表示本地主机。

4. `loop  txqueuelen 1  (Local Loopback)`

   - `loop`：接口类型为回环接口。
   - `txqueuelen 1`：发送队列长度为 1。
   - `(Local Loopback)`：本地回环接口。

5. `RX packets 2505  bytes 180228 (180.2 KB)`

   - `RX packets 2505`：接收数据包的数量为 2505。
   - `bytes 180228`：接收数据的字节数为 180228 字节（180.2 KB）。

6. `RX errors 0  dropped 0  overruns 0  frame 0`

   - `RX errors 0`：接收错误的数量为 0。
   - `dropped 0`：丢弃的接收数据包数量为 0。
   - `overruns 0`：接收数据超出缓冲区的次数为 0。
   - `frame 0`：帧错误的数量为 0。

7. `TX packets 2505  bytes 180228 (180.2 KB)`

   - `TX packets 2505`：发送数据包的数量为 2505。
   - `bytes 180228`：发送数据的字节数为 180228 字节（180.2 KB）。

8. `TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0`

   - `TX errors 0`：发送错误的数量为 0。
   - `dropped 0`：丢弃的发送数据包数量为 0。
   - `overruns 0`：发送数据超出缓冲区的次数为 0。
   - `carrier 0`：载波错误的数量为 0。
   - `collisions 0`：碰撞的数量为 0。



### 2 检测是否有WIFI网络

输入`sudo nmcli devide wifi list`，检测是否有wifi信号，若无，优先检测硬件情况，查看网卡的插拔情况是否正常，若正常，排除硬件问题，进行下一步。



### 3 排查软件方面的网络配置情况

1. **检查无线适配器状态:** 确保无线适配器已启用

   ```bash
   sudo nmcli radio wifi on
   #sudo nmcli radio wifi off #关闭无线网络
   ```

2. **刷新Network Manager:** 刷新Network Manager以确保其了解当前网络状态

   ```bash
   sudo service network-manager restart
   ```

3. **验证无线配置:** 仔细检查`wlan0`的无线配置。确保无线接口使用以下命令正确配置：

   ```bash
   sudo nmcli dev show wlan0
   ```

4. **扫描Wi-Fi网络:** 手动触发Wi-Fi扫描

   ```bash
   sudo nmcli device wifi rescan
   ```

5. **检查Wi-Fi网络列表:** 扫描后，使用以下命令检查可用的Wi-Fi网络：

   ```bash
   sudo nmcli device wifi list
   ```

6. **查看Network Manager日志:** 检查Network Manager日志是否有错误或相关信息：

   ```bash
   journalctl -xe | grep NetworkManager
   ```

7. **更新Network Manager:** 确保您的Network Manager 已更新至最新版本：

   ```bash
   sudo apt-get update
   sudo apt-get upgrade
   ```

8. **检查驱动程序问题:** 验证无线适配器的驱动程序是否已安装并正常工作。

	```bash
	sudo lshw -C network
	```



一般来说假如前三步都没问题，第四步手动扫描wifi时出现报错，例`Error: Scanning not allowed while unavailable or activating.`这个错误表明在无线网络设备不可用或正在激活时，无法执行扫描操作。则缩小范围，检测设备是否正常启动。



### 4 查看系统中的各种网络设备及其状态

#### 4.1 输入`nmcli device`，确保无线网络设备处于可用状态。

```shell
eth0    ethernet  connected  Wired connection 1  #以太网设备，已连接，使用有线连接1。
l4tbr0  bridge    unmanaged  --  #桥接设备，未受 NetworkManager 管理。   
dummy0  dummy     unmanaged  --  #虚拟设备，未受 NetworkManager 管理。  
rndis0  ethernet  unmanaged  --  #以太网设备，未受 NetworkManager 管理。  
usb0    ethernet  unmanaged  --  #以太网设备，未受 NetworkManager 管理。  
lo      loopback  unmanaged  --  #回环设备（loopback），未受 NetworkManager 管理。
wlan0   wifi      unmanaged  --  #无线设备，未受 NetworkManager 管理。
```

根据这些信息，`wlan0` 设备当前处于未受 NetworkManager 管理的状态，可能是由于之前在配置文件中指定了 `unmanaged-devices=interface-name:wlan0`。

#### 4.2 配置文件将 `wlan0` 设备纳入 NetworkManager 的管理

配置文件并删除或注释掉相关配置行以将 `wlan0` 设备纳入 NetworkManager 的管理，可以按照以下步骤进行操作：

1. 打开终端，并使用合适的文本编辑器（如 nano、vim 等）以管理员权限打开 NetworkManager 主配置文件。在大多数 Linux 发行版上，该文件位于 `/etc/NetworkManager/NetworkManager.conf`。

   例如，在终端中输入以下命令打开文件：

   ```
   sudo vim /etc/NetworkManager/NetworkManager.conf 
   ```

   将managed=false 
   改成managed=true

2. 在文本编辑器中，查找包含 `unmanaged-devices` 的行。该行可能类似于 `unmanaged-devices=interface-name:wlan0`。请注意，可能还有其他设备也被列为未受管理。

3. 要删除这一行配置，请将其完全从文件中删除。要注释掉这一行，可以在行的开头添加一个井号 `#`。

   示例：

   ```
   # unmanaged-devices=interface-name:wlan0
   ```

4. 保存更改（在 nano 编辑器中按下 `Ctrl + O`，然后按下 `Enter`），然后关闭编辑器（在 nano 编辑器中按下 `Ctrl + X`）。

5. 重新启动 NetworkManager 服务，以使更改生效。可以使用以下命令：

   ```
   sudo systemctl restart NetworkManager
   ```

6. 显示当前系统中的网络连接配置信息，查看已连接的网络
	```
	nmcli connection show 
	```



### 5 解决系统每次重启都会重置配置文件

系统在重启后自动生成了 `unmanaged-devices=interface-name:wlan0` 这段语句，可能是因为 NetworkManager 在重启时读取了默认的配置文件，并将之前的更改覆盖掉了。

为了解决这个问题，你可以尝试以下方法：

1. 创建一个 NetworkManager 配置文件片段：创建一个新的配置文件片段，其中包含你希望保留的配置项。在这个片段中，删除或注释掉 `unmanaged-devices=interface-name:wlan0` 这一行。例如，你可以创建一个名为 `/etc/NetworkManager/conf.d/99-custom.conf` 的文件。

   使用以下命令创建并编辑配置文件片段：

   ```
   复制代码sudo nano /etc/NetworkManager/conf.d/99-custom.conf
   ```

   在编辑器中添加以下内容：

   ```
   复制代码[keyfile]
   unmanaged-devices=none
   ```

   保存更改（在 nano 编辑器中按下 `Ctrl + O`，然后按下 `Enter`），然后关闭编辑器（在 nano 编辑器中按下 `Ctrl + X`）。

2. 重新启动 NetworkManager 服务：

   ```
   复制代码sudo systemctl restart NetworkManager
   ```

这样，当 NetworkManager 重启时，它将读取新的配置文件片段，并应该不再生成 `unmanaged-devices=interface-name:wlan0` 这段语句。

注意：确保新创建的配置文件片段位于 `/etc/NetworkManager/conf.d/` 目录中，这样 NetworkManager 在读取配置时会包含这个目录下的所有配置文件。

