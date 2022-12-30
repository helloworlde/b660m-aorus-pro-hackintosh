# 技嘉 B660M AORUS PRO AX 安装黑苹果

### 系统配置

| 类型      | 明细                                 |
| --------- | ------------------------------------ |
| 主板      | 技嘉 B660M AORUS PRO AX DDR4         |
| CPU       | 12th Gen Intel(R) Core(TM) i9-12900K |
| 内存      | 2 x Kingston 16GB 3200MHz DDR4       |
| 显卡      | 憾讯 AMD RX 6600 8GB                 |
| 硬盘      | Lexar 512G Nvme SSD                  |
| 声卡      | Realtek ALC897                       |
| 无线/蓝牙| Broadcom BCM943602CS, Intel Wi-Fi 6E AX210|
| 有线网口  | Intel Ethernet I-225V                |
| BIOS 版本 | F21                                  |

![ventura-info](./picture/ventura-info.png)

### 功能支持

|功能|状态|备注｜
|:--|:--|:---|
|CPU|✅|CPU型号能够正常识别，架构未识别，使用 Intel Power Gadget 可以跑满功率|
|内存|✅|运行频率正常，卡槽识别正常|
|核心显卡|❗|仅安装核心显卡时可以输出，但是非常卡顿|
|独立显卡|✅|免驱动的显卡安装后即可正常使用|
|WiFi & 蓝牙|⚠️|BCM943602CS 下功能均正常，WiFi协商速度为867Mbps，但是实际不到400Mbps；AX210 WiFi可以连接，但是无法切换到其他 WiFi，可能和驱动支持不完善有关，不能同时使用|
|网口|✅|协商速度 2.5G，实测接近|
|音频|✅|音频接口可以正常使用|
|USB|⚠️|USB 2.0/3.2 接口正常识别，Type C接口速度正常，部分接口未识别|
|散热|✅|正常识别风扇，散热正常|
|整机功率|⚠️|待机情况下功率较 Win 高50%|
|系统升级|✅|可以正常升级|
|iCloud相关|✅|可以正常使用|
|通用控制/AirDrop/屏幕镜像或扩展|✅|需要使用 BCM943602CS|
|睡眠/唤醒|⚠️|睡眠后无法重新点亮屏幕，硬件未休眠|

# 安装

## 1. 刻录 macOS 镜像

### 1.1 从 App Store 下载 Ventura 

在 App Store 或[在线的 App Store](https://www.apple.com/us/search/macOS?src=serp) 搜索 Ventura 并安装

![Download Ventura](./picture/download-ventura.png)

### 1.2 格式化并分区移动磁盘

使用磁盘工具格式化磁盘，选择抹掉，格式选择 "Mac OS 扩展（日志式）"；方案选择 GUID 分区图；这样就会将移动磁盘分为两个分区，一个是 EFI分区，另一个是系统镜像分区
![homelab-nuc-hackintosh-format-disk.png](https://hellowoodes.oss-cn-beijing.aliyuncs.com/picture/homelab-nuc-hackintosh-format-disk.png)

### 1.3 将 macOS 刻录到移动磁盘

下载完 macOS 安装器后，即可通过安装器将镜像刻录到移动磁盘；在命令行执行以下命令，Volume 即为刚才分区时命名的分区名称

```bash
sudo /Applications/Install\ macOS\ Ventura.app/Contents/Resources/createinstallmedia --volume /Volumes/macOS
```

等待执行完成即可

### 1.4 配置 EFI 

#### 修改配置信息

需要先修改 `config.plist` 文件中的配置

- CPU 信息

修改 `NVRAM > Add`下 `revcpuname` 的值为对应的 CPU，该配置仅用于`关于本机`显示，不修改会显示 iMac Pro 机型对应的 CPU

```xml
<dict>
	<key>revcpu</key>
	<integer>1</integer>
	<key>revcpuname</key>
	<string>16 Core Intel i9-12900K</string>
	<key>rtc-blacklist</key>
	<data></data>
</dict>
```

- 序列号信息

修改 `PlatformInfo > Generic` 对应的序列号信息，如果不使用 iCloud 账号及相关的功能，这部分可以不用配置；
可以使用 Hackintotool 获取，在`系统 > 序列号生成器` 选择设备为 `iMacPro1,1`， 然后生成；替换以下内容：

```xml
<key>Generic</key>
<dict>
	<key>AdviseFeatures</key>
	<false/>
	<key>MaxBIOSVersion</key>
	<false/>
	<key>MLB</key>
	<string>主板序列号</string>
	<key>ProcessorType</key>
	<integer>3841</integer>
	<key>ROM</key>
	<data></data>
	<key>SpoofVendor</key>
	<true/>
	<key>SystemMemoryStatus</key>
	<string>Auto</string>
	<key>SystemProductName</key>
	<string>iMacPro1,1</string>
	<key>SystemSerialNumber</key>
	<string>序列号</string>
	<key>SystemUUID</key>
	<string>SmUUID</string>
</dict>
```


#### 将 EFI 添加到移动磁盘的 EFI 分区

EFI 基于 [https://github.com/taruyato/b660m-aorus-pro-hackintosh](https://github.com/taruyato/b660m-aorus-pro-hackintosh) 修改而来；添加了 CPU，序列号等信息

首先将 EFI 分区挂载到电脑上；可以使用 `diskutil list` 命令查看 EFI 的 ID，然后使用命令行将 EFI 分区挂载

```bash
sudo mkdir /Volumes/efi
sudo mount -t msdos /dev/disk2s1 /Volumes/efi
```

## 2 安装 macOS

### 2.1 修改 BIOS 配置

建议先将 BIOS 升级到最新版本；进入配置后按 F7 将所有配置都恢复到默认之后再配置，避免因为其他配置被改动导致出现问题；选择高级模式进行配置：

| 菜单          | 功能                   | 选项                           | 值       | 备注 | 
|:------------ |:--------------------- |:----------------------------- | :------- | :--- |
| 频率/电压控制   | 进阶处理器设置           | Hyper-Threading 技术            | 启动     |      |
|               |                        | Intel 涡轮加速技术               | 启动     |      |
|               |                        | Legacy Game Compatibility Mode | 关闭     |      |
|               |                        | AVX		                   | 启动     |      |
|               | Extreme Memory Profile |                                | Profile1 |      |
| 设置           | IO Ports               | Internal Graphics              | 自动     | CPU有核显时配置  |
|               |                        | 4G以上解码                       | 启动     |      |
|               |                        | Re-Size BAR Support            |开启     |    如果使用6600系列显卡时开启  |
|               |                        | Super IO 配置 > Serial Port     | 启动     |      |
|               |                        | USB 程序 > XHCI Hand-off        | 开启     |      |
|               |                        | Network Stack Configuration > Network Stack | 关闭   |      |
|               |Miscellaneous            | Intel Platform Trust Technology(PTT)| 关闭     |  |
|               |                        | VT-d                           | 关闭     |      |
|               |                        | Trusted Computing > Security Device Support | 关闭     |      |
| 开机功能设置    |                        | CFG Lock                       | 关闭     |  |
|               |                        | 快速启动                         | 停用连接     |      |
|               |                        | Windows 10 功能                 | 其他操作系统     |      |
|               |                        | CSM 支持                         | 关闭     |      |
|               |                        | Secure Boot > Secure Boot       | 关闭     |      |


### 2.2 安装 macOS 

按照指引安装 macOS 即可，中途会重启多次；如果出现重启后找不到启动项，可以将移动硬盘拔掉重新插入再重启

等待安装完成，即可进入 macOS 配置，按照指引配置即可
