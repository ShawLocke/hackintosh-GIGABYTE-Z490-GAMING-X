# Win 10 Hackintosh 安装

自己根据官网制作 EFI，使用 ACPI 方式定制 USB，接近完美的 hackintosh。

主机已经出了，无法再更新了。记录留念。

## Windows 10专业版

1. 插入 U 盘，右键 U 盘，选择 "格式化"，点击 "还原设置的默认值"，选择文件系统为 "NTFS"，勾选 "快速格式化"，点击 "开始"
   > 文件系统不要用 FAT32（限制单个文件大小为 4G），也不要用exFAT（可能无法识别）

1. 按备注制作安装盘 或 下载镜像(<https://next.itellyou.cn/>, 选择 business edition)，然后用压缩软件 (比如 winrar)将镜像解压到 U 盘根目录 (右键镜像，选项 Open with WinRAR，选择 extract to，选择 U 盘根目录)
   > Windows下推荐用官方的`下载 Windows 10 光盘映像`制作安装盘
   > macOS下参考《Make Win Installer in macOS》制作安装盘

1. 重启电脑，按 Del 键或者快捷键（微星主板 F11，技嘉主板 F12，华硕主板F2）进入 BIOS，恢复默认值，改启动模式为“UEFI”（也就是disable CSM），改第一启动项选为“UEFI USB xxx”（鼠标悬停到图标上，如果看不到具体名称，说明主板没识别到 U 盘，拔插U盘或者插USB2.0接口），退出 BIOS，电脑自动重启

1. 安装时(可能遇到)"你选择的磁盘不在推荐的顺序"或"无法在该磁盘安装"等类似错误，这是由于 U 盘是 GPT 格式导致的（出现场景：在 macOS 中利用磁盘工具将 U 盘格式化为 GUID 格式了，然后再利用 Parallels Desktop 制作 Win10 启动盘，如果手头只有 macOS，不要这样制作启动盘，请参考《Make Win Installer in macOS》），解决方法：用 DiskGenius 将 U 盘删除整个分区，然后转换为 MBR 格式

1. 根据提示进行设置（尽量选择稍后设置）

1. 安装驱动

## GIGABYTE Z490 GAMING X 主板 BIOS

当前使用BIOS版本是：F21，官方下载下来的文件是 `mb_bios_z490-gaming-x_f21.zip`

升级或降级BIOS版本

1. 技嘉官网下载对应主板（**型号千万别弄错**）的BIOS版本
1. 下载下来的是压缩文件，解压出来放到C盘根目录；
1. 重启PC，按F12进入BIOS界面，找到Q-Flash
1. 按提示进行操作即可（**操作期间不可人为重启或断电，不然主板就得返修了**）

这个主板有bios救回方式：https://www.bilibili.com/video/BV1c84y1q7ns/

## Hackintosh准备 - USB定制

### **探测USB端口（Windows中操作）**

1. USBToolBox下载页面：https://github.com/USBToolBox/tool/releases，下载"Windows.exe"
1. 运行，提示安全隐患，不管它，还是同意运行，运行后会打开一个终端
1. 选择"D.  Discover Ports"，并回车
1. 依次在每个usb接口中分别插入usb2.0和usb3.0设备（记录下来），中间要等待5秒让软件识别

根据上述步骤，可以得出 "USB端口HS/SS关系表"（正对主板挡板，从上到下，从左到右，以技嘉z490 gaming x为例）：

> 注意：1个物理USB 3.0接口对应1个USB2逻辑接口 + 1个USB3逻辑接口
>
> 如何根据端口号推测出HS或SS后面的数字？
>
> USB2类型的端口号就是HS后面的数字，比如 9 -> HS09（2位数字），5 -> HS05
>
> SS后面的数字和与其相关联的USB2类型的端口号一样，比如 5 -> HS05, 21 -> SS05（不是21）

|PORT|USB类型|HS or SS|备注|
|-|-|-|-|
|10|USB 2|HS102|纯USB 2|
|9|USB 2|HS092|纯USB 2|
|5|USB 2|HS05|与下一个关联|
|21|USB 3|SS05||
|6|USB 2|HS06|与下一个关联|
|22|USB 3|SS06||
|1|USB 2|HS01|与下一个关联|
|17|USB 3|SS01||
|2|USB 2|HS02|与下一个关联|
|18|USB 3|SS02||
|3|USB 2|HS03|与下一个关联|
|19|USB 3|SS03||
|4|USB 2|HS04|与下一个关联|
|20|USB 3|SS04||
|12|USB 2 (蓝牙)|HS12|纯USB 2。在主板上，不在挡板上|
|13|USB 2 (ITE设备)|HS13|纯USB 2。在主板上，不在挡板上，看不到在哪里|

| Type   | Info                           | Comment |
|-|-|-|
|0x00| Type-A connector, USB 2.0 only | This is what macOS will default all ports to when no map is present. The physical connector is usually colored black|
|0x03| Type-A connector, USB 2.0 and USB 3.0 combined | USB 3.0, 3.1 and 3.2 ports share the same Type. Usually colored blue (USB 2.0/3.0) or red (USB 3.2)|
|0x08| Type C connector, USB 2.0 only | Mainly used in phones|
|0x09| Type C connector, USB 2.0 and USB 3.0 with Switch | Flipping the device does not change the ACPI port |
|0x0A| Type C connector, USB 2.0 and USB 3.0 w/o Switch |Flipping the device does change the ACPI port. generally seen on USB 3.1/3.2 mainboard headers|
|0xFF| Proprietary Connector | For Internal USB 2.0 ports like Bluetooth|

### **USB定制方法1 - ACPI** (定制好的文件存放在 `USB Mapping ACPI (GIGABYTE Z490 GAMING X)` 目录)

> 出处：<https://github.com/5T33Z0/OC-Little-Translated/tree/main/03_USB_Fixes/ACPI_Mapping_USB_Ports#port-mapping-options>，我根据自己的理解进行了调整和整理

> 优点：
> 1. 只针对 macOS 做对应修改，不影响其他 OS
> 1. 可以在 Windows 中全部完成（特别适合安装黑苹果前就定制好USB这种需求，当然也适合已经安装了黑苹果，想要自己定制USB那种需求）
> 1. 不与机型关联，可以随意更换机型
>
> 缺点：操作比较复杂，需要耐心（并不难）

1. Windows下将u盘格式化为 exFAT 或者 FAT 格式
1. 下载 <https://github.com/CloverHackyColor/CloverBootloader/releases>，选择 `CloverV2-xxxx.zip`，解压出来是 `CloverV2` 文件夹，将该文件夹下的所有文件拷贝到U盘
1. 重启，按 Del 键或者快捷键（微星主板 F11，技嘉主板 F12，华硕主板F2）进入 BIOS，选择 Clove 启动项（可能会有多个入口，选1个不对，就退出来，选其他，直到看到 Clove 引导页面），进入后，按一下 F4 键（这时按方向键会无反应，说明正在生成ACPI Tables），待可移动方向键后，退出 Clove
1. 电脑会自动重启，选择进入 Windows
1. 进入到 U盘，可以看到 `EFI\CLOVER\ACPI\origin` 下多出很多文件
1. 找到一个类似于 `SSDT-6-xh_cmsd4.aml` 名称的文件，用 <https://github.com/ic005k/Xiasl/releases>（跨平台，对应于macOS平台的 MaciASL，但 MaciASL 的批量缩进代码很麻烦）打开这个文件（注意看文件顶部内容）
   > Xiasl 打开 aml 文件时会自动生成一个 dsl 文件（aml是编译后的文件，dsl是对应的源文件）
1. 后续编辑 config.list 时, 在 ACPI > Delete 中添加一行规则：
   - All: False，好像 True 也行
   - Comment: Drop OEM USB Ports，这里按你自己意愿写即可
   - Enabled: True
   - OemTableID: 78685F636D736434 ，这是 xh_cmsd4 (OEM Table ID去掉两侧引号) 的HEX值 (ASCI to Hex converter转换)，注意：这里要根据你的实际情况来，不要照抄！
   - TableLength: 14689 （要填写十进制数字），而不是 14689 对应的HEX值 0x00003961。注意：这里要根据你的实际情况来，不要照抄！
   - TableSignature: 53534454 ，这是 SSDT 的 HEX值
1. 左侧导航栏中找到`\_SB.PCI0.XHC.RHUB > GUPC`
   ```
   # 修改 Method (GUPC … 这个方法为：
   Method (GUPC, 2, Serialized)
   {
      Name (PCKG, Package (0x04)
      {
            0xFF,
            0x03,
            Zero, 
            Zero
      })
      PCKG [Zero] = Arg0
      PCKG [One] = Arg1
      Return (PCKG)
   }
   ```
1. 我手头的技嘉Z490 GAMING X主板有26个端口，分别是HS01 ~ HS14, USR1 和 USR2, SS01 ~ SS10，根据上面的 "USB端口HS/SS关系表" 和 "USB类型HEX值表格" 来修改：

   ```
   # usb3.0接口（2个逻辑接口）： HS01 ~ HS06, SS01 ~ SS06，分别有1处要修改
   Method (_UPC, 0, NotSerialized)  
   {
      // 第1个参数 0xFF - 启用
      // 第2个参数 0x03 - Type-A connector, USB 2.0 and USB 3.0 combined
      // 注意第2个参数：HS01 ~ HS06 中也是 0x03，而不是 Zero
      Return (GUPC (0xFF, 0x03))
   }

   # 纯usb2.0接口：HS09, HS10，分别有1处要修改
   Method (_UPC, 0, NotSerialized)  
   {
      // 第1个参数 0xFF - 启用
      // 第2个参数 Zero（也可以是0x00）- Type-A connector, USB 2.0 only
      Return (GUPC (0xFF, Zero)) 
   }

   # 纯usb2.0接口（用于蓝牙）：HS12，有1处要修改
   Method (_UPC, 0, NotSerialized)  
   {
      // 第1个参数 0xFF - 启用
      // 第2个参数 0xFF - Proprietary Connector，也就是内置的意思
      Return (GUPC (0xFF, 0xFF)) // 蓝牙
   }

   # 没有用到的：HS07 HS08, HS11, HS13 HS14, SS07 ~ SS10，分别有2处要修改
   Method (_UPC, 0, NotSerialized)  
   {
      If (_OSI ("Darwin")) // 如果是苹果系统
      {	
         // 第1个参数 Zero - 禁用，第二个参数填什么都不重要了
         Return (GUPC (Zero, Zero))
      }
      Else
      {
         // 其他系统，不受影响
         Return (GUPC (0xFF, 0x03))
      }
   }

   Method (_PLD, 0, NotSerialized)
   {
      // xxx 是这个方法的原有方法体，xxx是以if打头的
      If (_OSI ("Darwin"))
      {
         Return (GPLD (Zero, Zero))
      }
      Else xxx
      

      // 也可以按下面这种方式来，和上面是等价的，这种方式dsl编译出来的aml会是上面的形式
      If (_OSI ("Darwin")) // 如果是苹果系统
      {
         Return (GPLD (Zero, Zero))
      }    
      Else
      {
         // Else中放原有代码
      }
   }

   # USR1, USR2，macOS不支持，分别有2处要修改
   Method (_UPC, 0, NotSerialized)
	{
      Return (GUPC (Zero, Zero)
   }
	
	Method (_PLD, 0, NotSerialized)
	{
		Return (GPLD (Zero, Zero))
   }
   ```

1. 保存（所有修改都是保存在dsl文件中），然后点击顶部菜单栏中的edit > complie，就会在同目录下生成一个aml文件，改名成 "SSDT-PORTS"（或者你容易理解的名字）
   > Xiasl 打开 aml 文件时会自动生成一个 dsl 文件（aml是编译后的文件，dsl是对应的源文件），所以等所有事情完成后，可以删除 dsl 文件，只需要保留 aml 文件
1. 将上述两个kext文件加入到"EFI/OC/ACPI"文件夹中，并修改"config.list"将它添加进来并启用
   > 如果文件夹中原来就有RHUB.aml等文件，记得禁用它们

### **USB定制方法2 - KEXT**

> 优点：操作简单
>
> 缺点：
> 1. Windows中操作完，macOS中还得利用 Hackintool (<https://github.com/headkaze/Hackintool>) 生成 `USBPorts.kext`
> 1. 与机型关联起来了，换机型后需要修改 Hackintool 生成的 `USBMap.kext`（有2个地方涉及到机型）

1. USBToolBox下载页面：<https://github.com/USBToolBox/tool/releases>，下载"Windows.exe"
1. 运行，提示安全隐患，不管它，还是同意运行，运行后会打开一个终端
1. 选择"D.  Discover Ports"，并回车
1. 依次在每个usb接口中分别插入usb2.0和usb3.0设备（记录下来），中间要等待5秒让软件识别
1. 输入"B"，并回车
1. 选择"S.  Select Ports and Build Kext"，并回车
1. 已选条目是绿色，未选条目是白色
    - 将ITE Device连接的端口改为未选，输入端口号（比如"13"），然后回车。结果：对应的条目变为白色
    - 将Bluetooth连接的端口类型改为255，输入T:12:255（我这里的端口号是12，你根据情况做相应修改），然后回车。结果：对应的条目的端口类型变为Internal
    - 发现某个usb2接口（接口颜色黑色）也绑定了2个端口，有必要将多余的端口改为未选:
	   - 修改设置：输入"B"，并回车，选择"C.  Change Settings"，并回车，选择"C.  Bind Companions"，并回车，发现C.  Bind Companions的状态变为"Disabled"后，按"B"，并回车，选择"S.  Select Ports and Build Kext"，并回车
	   - 输入要改为未选的端口号，比如"25,26"，并回车。结果：25和26端口变为白色
	   - 然后将25,26绑定的9,10的端口类型改为0，输入"T:9,10:0"，并回车。结果：9和10端口类型变为Type A
1. 请根据具体情况调整，最终确保端口总数 <= 15
1. 选择"K. Build UTBMap.kext (requires USBToolBox.kext)"，并回车，会生成"UTBMap.kext"文件
1. 然后下载 <https://github.com/USBToolBox/kext/releases>，选择"USBToolBox-1.1.1-RELEASE.zip"，解压找到"USBToolBox.kext"文件
1. 将上述两个kext文件放到 `EFI/OC/Kexts` 文件夹中，用ProperTree (<https://github.com/corpnewt/ProperTree>)修改 config.list（选择菜单栏中的 `File/OC Clean Snapshot`，保存）
1. 进入macOS后，使用 Hackintool 检查定制是否达到预期（应该是没有问题的，上面已经将蓝牙内置了），导出 USBPorts.kext 文件
1. 将 USBPorts.kext 放到 `EFI/OC/Kexts` 文件夹中，并将上面两个 kext 文件 删除，用ProperTree修改 config.list（选择菜单栏中的 `File/OC Clean Snapshot`，保存）

## Hackintosh准备 - EFI制作

> - 自己制作，参考：[OpenCore文档](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/opencore-efi.html) 和 <<https://dortania.github.io/docs/>>
> - 直接用其他人制作的 **同主板** EFI
>
> 所需软件：
> 1. [ProperTree](https://github.com/corpnewt/ProperTree)，用来编辑config.plist, 不推荐其他编辑器，比如OpenCore Configurator (OCC)或OCAT，因为它们可能会无故多一些或者少一些内容
> 1. [OCAT (OpenCore Auxiliary Tools)](https://github.com/ic005k/OCAuxiliaryTools)，用来生成三码

1. 使用样本EFI，删除部分文件
   > 对应于 <https://dortania.github.io/OpenCore-Install-Guide/installer-guide/opencore-efi.html>
   - 下载[OpenCorePkg](https://github.com/acidanthera/OpenCorePkg/releases)，下载RELEASE版本，解压，进入 `OpenCore-x.x.x-RELEASE/X64/`，拷贝 EFI 文件夹到某一位置
   - `EFI/OC/Drivers/` 中保留: `OpenRuntime.efi`（必须）和 `ResetNvramEntry.efi`（reset NVRAM用），其他全部删除
   - `EFI/OC/Tools/` 中保留: `OpenShell.efi`（调试用），其他全部删除
1. 收集Drivers和Kexts
   > 对应于 <https://dortania.github.io/OpenCore-Install-Guide/ktext.html#firmware-drivers>
   - `EFI/OC/Drivers/` 中添加：[HfsPlus.efi](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi)：必须
   - `EFI/OC/Kexts/` 中添加：
      > 都用release版本
      > 官方说用ProperTree加载下面这些kexts不用管顺序，原文是：Here's where we specify which kexts to load, in what specific order to load, and what architectures each kext is meant for. By default we recommend leaving what ProperTree has done, however for 32-bit CPUs please see below
      - Must Haves
         - [Lilu.kext](https://github.com/acidanthera/Lilu/releases) : 必须，其他很多kext依赖它
         - [VirtualSMC.kext](https://github.com/acidanthera/VirtualSMC/releases) : 必须，模拟真实mac电脑的SMC
      - 图像
         - [WhateverGreen.kext](https://github.com/acidanthera/WhateverGreen/releases) :必须
      - 音频
         - [AppleALC.kext](https://github.com/acidanthera/AppleALC/releases) : 必须
      - Ethenet
         - [IntelMausi.kext](https://github.com/acidanthera/IntelMausi/releases) : 必须
         > 其他主板可能不是用这个kext，如何查看自己主板的Ethenet型号：使用AIDA64（下载安装图吧工具箱），看计算机/系统概述/网络设备/网络适配器，我这里显示是Intel(R) Ethernet Connection (11) I219-V，然后在上面页面中可以搜索"219"，可以判断出使用IntelMausi合适
      - USB
         - 建议在安装前参考"Hackintosh准备 - USB定制"提前定制好USB（推荐用ACPI方式定制，即使用KEXT方式USB定制，也不需要用到 `XHCI-unsupported.kext`）
      - WiFi and Bluetooth
         - 建议用白果卡，比如T919（也就是94360CD）
      - 其他
         - [NVMeFix.kext](https://github.com/acidanthera/NVMeFix/releases) : 建议
         - [RestrictEvents.kext](https://github.com/acidanthera/RestrictEvents/releases) : MacPro 机型记得带上，主要用于消除 MacPro 机型每次启动的内存警告提示
1. 收集SSDTs
   > 对应于 <https://dortania.github.io/Getting-Started-With-ACPI/ssdt-platform.html>
   - `EFI/OC/ACPI/` 中添加:
      > 推荐使用工具 SSDTTime (Window下运行) 来制作，也可以直接用编译好的通用aml文件，也可以手工制作
      > 需要用到哪些？根据CPU所属的平台来决定，比如10700属于Comet Lake平台（搜索10700 Code Name）
      - CPU (必须)：[SSDT-PLUG](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-PLUG-DRTNIA.aml)
      - EC (必须)：[SSDT-EC-USBX](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-EC-USBX-DESKTOP.aml)
      - AWAC (必须)：[SSDT-AWAC](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-AWAC.aml)
      - USB (非必须)：[SSDT-RHUB](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-RHUB.aml)
         > 只有华硕某些主板需要 `SSDT-RHUB`，如果用ACPI方式定制USB不要加这个，可能会导致ACPI方式定制USB无法生效
1. 设置config.plist
   > 对应于 <https://dortania.github.io/OpenCore-Install-Guide/config.plist/#creating-your-config-plist>
   - 从 `OpenCore-x.x.x-RELEASE/Docs/` 中拷贝 `Sample.plist` 到 `EFI/OC/` 下，并重命名为 `config.plist`
   - 使用 ProperTree 打开 `config.plist`，然后选择菜单栏中的 `File/OC Clean Snapshot`，在弹出窗口中，指向 `EFI/OC`这个文件夹，保存
      > 当修改到 ACPI, Drivers 或 Kexts 时，一定要记得OC Clean Snapshot一下
   - 将 "Hackintosh准备 - USB定制" 出来的 aml 文件放到 `EFI/OC/ACPI/` 下，选择菜单栏中的 `File/OC Clean Snapshot`，然后选中 `EFI/OC/ACPI/Delete/1`，右键，选择 Copy，然后选中 `EFI/OC/ACPI/Delete`，右键，选择 Paste，参考 "后续编辑 config.list 时, 在 ACPI > Delete 中添加一行规则" 那一段进行修改，保存
      > 如果用kext方式的USB定制，那就将文件放到 `EFI/OC/Kext/` 即可，不需要加 ACPI > Delete 规则。**注意**：这种方式与机型相关
   - 参考 <https://dortania.github.io/OpenCore-Install-Guide/config.plist/#selecting-your-platform>，选择平台，按照指南进行编辑
      > 详尽解释参考 <https://dortania.github.io/docs/>，问题排查参考 <https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/troubleshooting.html#table-of-contents>
      > 这里只是列出部分需要关注的内容，其他按照指南做即可
      - `Booter > Quirks`: 这一部分对启动很重要，early boot failure基本上是这里的设置导致的。请细看官网文档，弄清楚你的主板是否需要特殊设置
      - `DeviceProperties/Add/PciRoot(0x0)/Pci(0x2,0x0)`: 只有独显记得不需要这个
      - `Misc/Boot/ShowPicker`: True - 显示 "启动选择菜单页面" ，False - 不显示（启动时可以按 Alt 或者 Option 键让其显示出来）
      - `Misc/Boot/HideAuxiliary`: True - 在 "启动选择菜单页面" 中按 空格键 出现辅助等工具，False - 让它们总是出现
      - `Misc/Security/AllowSetDefault`: 建议设置为 True，在 "启动选择菜单页面" 中用方向键移动到某个选项，然后按 `ctrl+enter`（也可以用ctrl+选项前的索引号一个步骤），以后这个选择就会成为默认选项
      - `Misc/Debug`: 不要参考文档做（文档建议启用），保持默认即可，不然 EFI 文件夹中会多出很多日志文件
      - `NVRAM/Add/7C436110-AB2A-4BBB-A880-FE41995C9F82/` 
         - `boot-args`: Navi GPUs (RX 5000 & 6000 series)需要加 `agdpmod=pikera`，其他显卡不要加！
         - `prev-lang:kbd`: 默认是俄文，建议改为 `656e2d55533a30`（也就是 en-US ）
      - `PlatformInfo/Generic`
         - `SystemProductName`: 10代机型推荐，iMac20,1 (10700K及以下), iMac20,2 (i7-10700K以上)
         - `SystemSerialNumber`, `MLB`, `SystemSerialNumber`: 运行 OCAT，选择导航栏中的 PI，选择机型，生成，将生成的三项填写进去
            > 打开[Apple's Check Coverage Page](https://checkcoverage.apple.com/)页面，输入SN，检查，页面如果反馈是 invalid就对了，这是我们需要的
         - `ROM`: 先用OCAT随机生成的，安装完成后参考 [Fixing iServices](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html) 修改
         - `ProcessorType`: 填为0（自动读取CPU型号，解决"关于本机"中处理器显示不正确或者未知的问题）

## Hackintosh安装

> 所需软件：
>
> 1. [balenaEtcher](https://www.balena.io/etcher/)，用来烧录镜像
> 1. [DiskGenius](https://www.diskgenius.com/)，只有Windows版，请用中文版，免费版即可

1. 下载镜像
   > 可以自己制作镜像（在macOS中才能完成所有流程），参考：《安装镜像 - 利用gibMacOS工具制作原版dmg安装镜像教程.html》
   - <https://bbs.pcbeta.com/viewthread-1939091-1-2.html>（远景论坛 小7编辑员，纯净版镜像，如果不想自己制作镜像，就用这个）
   - <https://blog.daliansky.net/categories/%E4%B8%8B%E8%BD%BD/%E9%95%9C%E5%83%8F/>（黑果小兵，可能需要微信打赏才能得到最新系统的下载）

1. 插入 U 盘到USB 3.O接口（写入速度更快），将U盘格式化，点击“还原设置的默认值”，勾选“快速格式化”，然后使用 balenaEtcher 制作安装盘到 U 盘，进行中或者完成后可能会提示“需要格式化 U 盘...”，一律选择“取消”
   > 如果之前制作过安装盘，想要重新制作，用 DiskGenius 清除扇区数据，看到运行够 5 秒后就点击“取消”（不要去等待完成，需要很长时间），然后格式化U盘
   > GitHub上有人反馈balenaEtcher总是flash failed的情况，我也遇到过，好像与Windows安全中心有关，后来用火绒替代Windows中心后才正常
   > Windows中制作安装盘一直失败，如果有macOS电脑，可以在macOS中制作安装盘：1. 用系统自带的disk utility格式化u盘，格式选为扩展（日志式），方案选为GUID. 2. 使用balenaEtcher烧录macOS镜像。**注意**：不要在macOS中用balenaEtcher制作安装盘后又拿到Windows下来修改，不然后续安装黑苹果可能会出问题，也就是说不要混用macOS和Windows制作的安装盘

1. 使用 DiskGenius，左侧导航栏中的 U 盘，左侧导航栏中选择“EFI”分区，右侧上方 Tab 选择“浏览文件”，然后全选所有文件，右键“强制删除”，然后将 EFI 文件夹拖入到 EFI 分区
   > 如果是macOS中制作的安装盘，在终端中输入“sudo diskutil list”，找到U盘的EFI分区的identifier，然后 sudo diskutil mount [identifier]，桌面会自动出现 EFI 分区，双击进入，删除里面的所有文件，然后将上一步编辑好的 EFI 文件夹拖进来

1. 重启电脑，按 Del 键或者快捷键（微星主板 F11，技嘉主板 F12，华硕主板F2）进入 BIOS，先恢复默认值，再设置：
   > 详细设置参考 <https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html#intel-bios-settings>（这里是以comet-lake平台举例）
   - Intel VT-x (虚拟化技术) [允许]
   - Intel VT-d [禁止]（如果 config.list 中设置 `DisableIoMapper` 为 True, 那这里可以不管）
   - CFG Lock [禁止]（如果 config.list 中设置 `AppleXcpmCfgLock` 为 True, 那这里可以不管）
   - (推荐) Fast Boot [禁止], Secure Boot [禁止]
   - 改第一启动项为“UEFI USB xxx”（鼠标悬停到图标上，如果看不到具体名称，说明主板没识别到 U 盘，拔插 U 盘或者插 USB2.0 接口）

1. 在引导菜单界面，选择“Install ...”，出现 Apple Logo 和进度条，耐心等待完成（如果进度条跑了一会出现禁止标志，换用USB2.0接口试试）
   > 特别注意：集显/独显/集显+独显是否设置正确，机型与三码是否匹配，这些很容易让这一步出问题

1. 在安装界面，选择“磁盘工具”，点击继续进入磁盘工具界面，点击左上角的“显示”，选择“显示所有设备”，选择对应的磁盘，点击上方的“抹掉”，填写名称：hac（按你意愿填写即可），选择格式“APFS”，选择方案“GUID 分区图”，点击“抹掉”，等待处理完后，点击“完成”，再点击左上角的“x”退出磁盘工具，自动回到安装界面

1. 安装界面，选择“安装 macOS”（如果待安装磁盘不可用，将鼠标悬停到磁盘图标上，能看到提示说磁盘需要格式化为 APFS，那就回到磁盘工具界面去处理），然后一路的点击“同意/继续”
   > 这一步到下一步中间可能会有多次自动重启，前几次应该会自动选择Installer（如果没有自动选择，记得手动选择），最后一次应该会自动选择之前设置的磁盘名（如果没有自动选择，手动选择）

1. 欢迎界面，大多是一些设置（尽量选择稍后设置，记得先不要登录苹果账号），设置完成后就会进入系统了

1. 进入系统后，系统会提醒设置键盘，根据提示操作即可，这里顺带设置：
   - 系统偏好设置：鼠标 -> 反勾选“滚动方式：自然”（让鼠标滚动方式与 Windows 保持一致）
   - 键盘 -> 修饰键 -> 选择键盘中选“USB键盘”(如果是优联键盘，不要用默认那个 USB Receiver这是鼠标用的，要选择USB Receiver 1，不然后面的键位调换在重启后会失效) -> Option 键和 Command 键对调（让按键与 Apple 原生键盘保持一致），输入法 -> 简体拼音 -> 反勾选“使用大写锁定键切换“ABC”输入模式”（让输入大写方式与 Windows 保持一致）

1. 挂载 U 盘中的 EFI 分区，在终端中输入“sudo diskutil list”，找到U盘的EFI分区的identifier，然后 sudo diskutil mount [identifier]，桌面会自动出现 EFI 分区，双击进入，将其中的EFI文件夹拖到桌面

1. 利用终端挂载磁盘 EFI 分区（访达中自动出现 EFI 分区），将桌面中的 EFI 文件夹拖到访达中的 EFI 分区里（如遇冲突，选择替换，或者先删除已存在的EFI，然后再拖入新的EFI）

1. 拔掉 U 盘，重启电脑，按 Del 键或者快捷键（微星主板 F11，技嘉主板 F12，华硕主板F2）进入 BIOS，将 UEFI Hard Disk xxx 设为第一启动项(macOS 所在的 SSD)，退出 BIOS，电脑会自动重启

   > 有些主板可能看不到 UEFI Hard Disk xxx，比如微星 B460M 迫击炮，可以去“Settings/Boot/UEFI Hard Disk Drive BBS Priorities"中设置

## Hackintosh安装后的工作

1. [Post-install](https://dortania.github.io/OpenCore-Post-Install/)（处理好的文件存放在 `Fixing keyboard wake issues` 目录）
   > 主要是下面几个，做完记得重启，reset NVRAM 生效
   - [Fixing Audio](https://dortania.github.io/OpenCore-Post-Install/universal/audio.html): 声音相关
   - [Fixing iServices](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html): ROM 相关
   - [Fixing Instant Wake](https://dortania.github.io/OpenCore-Post-Install/usb/misc/instant-wake.html): 睡眠随机唤醒等情况
      > 遇到到有这个就会导致无法用键盘或鼠标唤醒（可以按电源键唤醒），但后来再没出现
      - look through your DSDT and see what you have 怎么理解？在 windows 中用 SSDTTime 去生成 DSDT，用 Xiasl 打开 DSDT.aml 文件，再参考 OC 文档处理选择用哪个 aml
      - 注意：OC 文档中链接打开的 ACPI Patch，在 github 网页或文本编辑器比如 VS Code 中显示 data 类型值并不是 HEX 值，它是被 base64 encoded 的值，举例: `R1BSVwI=` base64 decode 之后是 `GPRW `(注意最后有个空格)，然后利用 `ASCII Text to Hex Code` 可以得到 `4750525702`（HEX值，将这个值填到 config.list中去）
   - [Fixing Keyboard Wake Issues](https://dortania.github.io/OpenCore-Post-Install/usb/misc/keyboard.html): 键盘或鼠标唤醒时需要多按几次显示器才能亮屏的情况。我用的是 "Create a fake ACPI Device" （重启并reset nvram生效），官方推荐的 "Add Wake Type Property" 没效果
      > 这个不做也可以，多按几下键盘或者鼠标唤醒，或者按电源键唤醒
      - 下载 `USBWakeFixup.kext`: Both under `EFI/OC/Kexts` and `config.plist`
      - 打开 `SSDT-USBW.dsl`，将里面的内容复制，打开 Xiasl，新建一个文件，将复制的内容拷贝进去
      - 下载并运行 `gfxutil`，结果中找 `XHC, XHC1 或 XHCI`，我找到的是 `/PCI0@0/XHC@14 = PciRoot(0x0)/Pci(0x14,0x0)`，这里要做一个转换：`/PCI0@0/XHC@14` -> `\_SB.PCI0.XHC`
      - Xiasl 中新建的那个文件有2个地方（`\_SB.PCI0.XHC._PRW`）可能需要修改，由于我这里上一步的转换和模板一样，就不用修改了
      - 保存，文件名填写为 `SSDT-USBW`，会在你指定的目录下生成 `SSDT-USBW.dsl`
      - 点击导航栏 `Edit > Compile`，会在上面的同目录下生成 `SSDT-USBW.aml`: Both under `EFI/OC/ACPI` and `config.plist`
      - 重启，reset NVRAM生效

1. 检查
   - 关机，重启，睡眠及唤醒
   - WIFI和蓝牙，airdrop空投, handoff接力, iMessage, FaceTime, iCloud, App Store等
   - USB接口，板载网卡，板载声卡等
   - 睿频（intel power gadget，菜单中的test）
   - 显示（核显及独显）：利用VideoProc Converter 4K查看硬解4k情况
   
---

1. 睡眠唤醒后蓝牙有时候不可用？
   > 表现形式有：键盘唤醒时键盘可用，鼠标不可用，反之类似，电源键唤醒，键盘和鼠标不可用，进到蓝牙设置里发现搜寻图标闪烁得和正常情况不一样，等一段时间后（可能30s）才正常，甚至可能要重启才能恢复
   > 按下面的方式处理后，就无法用蓝牙鼠标或者键盘唤醒睡眠了，只能用 USB 鼠标或者键盘唤醒，或者用电源键唤醒
   > 参考：<https://zhuanlan.zhihu.com/p/79246309>
   ```shell
   # 安装 sleepwatcher
   brew install sleepwatcher

   # 安装 blueutil，注意：使用 sudo pkill bluetoothd 无效，利用 blueutil 有效
   brew install blueutil

   # 在根目录下新建 2 个文件
   cd ~
   touch .sleep
   touch .wakeup

   # 授予 read, write, and execute 权限 for everyone
   chmod 777 .sleep 
   chmod 777 .wakeup 

   # 查看 blueutil 路径，比如 /usr/local/bin/blueutil，后面用得到
   which blueutil

   # 利用 vim (vim .sleep) 往 .sleep 中写入内容，睡眠时关闭蓝牙
   /usr/local/bin/blueutil -p 0
   echo "[`date "+%Y-%m-%d %H:%M:%S"`] sleep $?" >> ~/.sleepwatcher.log

   # 利用 vim (vim .wakeup) 往 .wakeup 中写入内容，唤醒时开启蓝牙
   /usr/local/bin/blueutil -p 1
   echo "[`date "+%Y-%m-%d %H:%M:%S"`] wakeup $?" >> ~/.sleepwatcher.log

   # 运行服务，会弹窗要求一个系统权限，即：在 Privacy & Security > Input Monitoring 中添加 sleepwatcher，同意
   brew services start sleepwatcher

   # 查看进程，检查执行脚本的路径对不对，应该是 /Users/xxx/.sleep 和 /Users/xxx/.wakeup ，xxx 是用户名。
   # 如果路径不对，你会发现没有日志，因为脚本根本没有执行
   ps aux | grep sleepwatcher

   #### 如果是 /Users/brew/actions-runner/_work/homebrew-core/homebrew-core/bottles/home/，这就
   ### 参考 <https://github.com/Homebrew/discussions/discussions/4231>
   ## 1. 重装 sleepwatcher
   HOMEBREW_NO_INSTALL_FROM_API=1 brew reinstall sleepwatcher
   ## 2. 重启服务
   brew services restart sleepwatcher
   ## 3. 查看进程，检查执行脚本的路径对不对
   ps aux | grep sleepwatcher

   # 查看 ~/.sleepwatcher.log 来验证是否执行了脚本，如果没有问题了，删除输出日志的命令和 .sleepwatcher.log
   ```

1. 睡眠有问题？
   > 参考：<https://dortania.github.io/OpenCore-Post-Install/universal/sleep.html#preparations>，如果还是不行就放弃吧，参考<https://sspai.com/post/61379>设置（台式机没有睡眠也能接受）
   - in macOS
      - terminal中运行
         ```shell
         # Disables autopoweroff: This is a form of hibernation
         sudo pmset autopoweroff 0
         # Disables powernap: Used to periodically wake the machine for network, and updates(but not the display)
         sudo pmset powernap 0
         # Disables standby: Used as a time period between sleep and going into hibernation
         sudo pmset standby 0
         # Disables wake from iPhone/Watch: Specifically when your iPhone or Apple Watch come near, the machine will wake
         sudo pmset proximitywake 0
         # Disables TCP Keep Alive mechanism to prevent wake ups every 2 hours
         sudo pmset tcpkeepalive 0
         ```
      - 补充：关闭 tty, network activity 等
         > 参考 <https://github.com/rafaelmaeuer/Asus-Z590-P-Hackintosh/blob/main/Docs/SLEEP.md>
         - 关闭 Energy Saver 中的 `Enable Power Nap` (也就是 pmset 中的 `powernap`)
         - 关闭 Energy Saver 中的  `Wake for network access` (也就是 pmset 中的 `womp`，它和 Wake on LAN 有关系，所以 BIOS 中关闭 Wake on LAN)
         - 终端中运行：`sudo pmset ttyskeepawake 0`
         - 如果睡眠被唤醒，比如：`Darkwake from Normal Sleep [CDN]: due to RTC PEG2 PEG3 PR04/Maintainance`，解决方法：启用 `Kernel > Patch` 中的 `Disable RTC wake scheduling` 条目，记得 reset NVRAM，注意：不要去 `boot-args` 中添加 `darkwake=0`
   - in config.plist:
      While minimal changes are needed, here are the ones we care about:
      - `Misc -> Boot -> HibernateMode -> None`
         - We're gonna avoid the black magic that is S4 for this guide
      - `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args`
         - `keepsyms=1` - Makes sure that if a kernel panic does happen during sleep, that we get all the important bits from it
         - `swd_panic=1` - Avoids issue where going to sleep results in a reboot, this should instead give us a kernel panic log
   - in BIOS
      - disable:
         - Wake on LAN: 技嘉z490 gaming x主板 `Settings > IO Ports > Wake on LAN Enable`
         - Trusted Platform Module: 技嘉z490 gaming x主板 `Settings > Miscellaneous > Intel Platform Trust Technology (PTT)`
         - Wake on USB: 技嘉z490 gaming x主板没找不到
      - enable:
         - Wake on Bluetooth(If using a Bluetooth device for waking like a keyboard, otherwise you can disable): 技嘉z490 gaming x主板没找到

1. 关机变为重启？如果USB定制好了，一般不会出现这种情况
   - <https://dortania.github.io/OpenCore-Post-Install/usb/misc/shutdown.html>
      > 看不懂？参考 <https://github.com/Koala166/The-TLDR-Guide-of-Fixing-Shutdown-Restart>

1. 如果是双系统，切换到 Windows，发现时间错误
   - 管理员身份运行 cmd
   - 运行命令：Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1 （删除方法是：计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation 中删除 RealTimeIsUniversal 这一项）
   - 重启电脑生效
   
1. warning after boot: 'You Shut Down Your Computer Because of A Problem'
   1. Search Console in Spotlight.
   1. Choose DiagnosticReports in left nav bar.
   1. Type "sleep" in the top-right search bar.
   1. Select DiagnosticReports, Look for files named Sleep Wake Failure. 
   1. Right-click on the files and choose "Move to Trash".
   1. Reboot.

1. 黑苹果电脑名与关于本机概览中的型号不一致（EFI中换机型导致的）
   - 系统偏好 - 共享 - 电脑名称，修改为你想要的，比如ming's iMac
   - 系统偏好 - 共享 - 电脑名称的右下侧方的Edit按钮，Local Hostname修改为mings-iMac (和上方的区别是'去掉了，空格变为-)

1. 安装过黑苹果的主机可能会出现无法进入BIOS的情况
   - 主机断电，拔掉电源线
   - 拿出BIOS电池，等待15s，再装回
   - 重新开机

1. 双硬盘双系统，Reset NVRAM后黑苹果引导不见了，只有Windows引导
   - 进入Windows系统，打开EasyUEFI
   - 删除黑苹果启动项（如果有的话）
   - 新建一个启动项，`Type`选为`Linux or other OS`, `Description`填为`Mac`（按你的意愿来填写），选择EFI所在的分区，`File path`选为`EFI/BOOT/BOOTx64.efi`, 点击OK
   - 将新建的启动项移动到第一位
   - 关闭EasyUEFI，重启
   - 进入BIOS，设置硬盘优先级，将黑苹果硬盘放到第一位，保存，重启

1. 先装了Windows，然后装了黑苹果，后来拆了Windows硬盘，主机无法启动
   - 装回Windows硬盘，想办法正常进入黑苹果
   - 找到黑苹果的EFI分区，删除里面的Microsoft文件夹
   - 拔掉Windows硬盘，重置nvram即可
   > 如果Windows硬盘已经不在了，想办法用U盘启动，然后进行黑苹果后再替换EFI

---

1. 装机 <https://www.bilibili.com/video/av37032107?from=search&seid=9274019570257299236>
1. NVME SSD 安装 <https://www.bilibili.com/video/av45249255/?spm_id_from=333.788.videocard.1>
   > 显卡厚度超过2插槽时，无线网卡就只能插到最底下的pcie插槽，但这时如果用了第二个m2接口，会发现无线网卡连接到的主板usb插槽无法工作（蓝牙无法使用），这是主板的限制。如果显卡厚度超过2插槽，最好不要用第二条m2，改用SATA固态硬盘。
1. EFI
   - <https://github.com/GeQ1an/MSI-B360M-MORTAR-HACKINTOSH-OPENCORE-EFI> (MSI B360M 迫击炮，曾经用过同主板)
      > README 写得很好，有些内容可以参考
   - <https://github.com/5T33Z0/Gigabyte-Z490-Vision-G-Hackintosh-OpenCore>
      > 同为技嘉Z490系列，README 写得很好
   - <https://github.com/SchmockLord/Hackintosh-Intel-i9-10900k-Gigabyte-Z490-Vision-D>
   - <https://github.com/SuperNG6/MSI-B660-Monterey-EFI> (MSI B660M 迫击炮，以后可能用得上)