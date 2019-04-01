# macOS 安装用EFI

- 1-23-2019

  - 更新CLOVER到v4859
  - 跟随`macOS Mojave 10.14.3(18D42)`发布

- 1-20-2019

  - **1-20-2019更新：10.14.x解除USB端口限制补丁**

    ```ruby
    Comment: USB port limit patch 10.14.x modify by DalianSky(credit ydeng)
    Name: com.apple.iokit.IOUSBHostFamily
    Find: 83FB0F0F
    Replace: 83FB3F0F
    MatchOS: 10.14.x
    
    Comment: USB Port limit patch 10.14.x modify by DalianSky(credits PMheart)
    Name: com.apple.driver.usb.AppleUSBXHCI
    Find: 83FB0F0F
    Replace: 83FB3F0F
    MatchOS: 10.14.x
    ```

    

- 1-11-2019

  - CLOVER更新到v4844
  - 驱动更新
    - Lilu v1.3.2
    - AppleALC v1.3.5
    - Whatevergreen v1.2.7
    - USBInjectALL v0.7.1
  - 添加`config_UHD630_4K_HBR2_3E9B0000`配置文件
  - 添加`config_UHD630_1080P_HBR_3E9B0000`配置文件
  - 添加`config_WHISKEYLAKE_3EA0`配置文件



## UHD 630 `divide error`由于在帧缓冲驱动程序中被除零而导致内核恐慌的几乎终极解决方案

```
Anonymous UUID: A6C52317-1B2D-E00D-241C-DBCE7C091990

Sat Sep 29 13:17:09 2018

*** Panic Report ***
panic(cpu 8 caller 0xffffff80004d781d): Kernel trap at 0xffffff7f837537d0, type 0=divide error, registers:
CR0: 0x0000000080010033, CR2: 0xffffff81f645e000, CR3: 0x00000004512ce05c, CR4: 0x00000000003626e0
RAX: 0x017d68fdc0000000, RBX: 0x017d68fdc0000000, RCX: 0x0100000100000000, RDX: 0x0000000000000000
RSP: 0xffffff81e8ba3540, RBP: 0xffffff81e8ba3570, RSI: 0xffffff81e8ba3388, RDI: 0xffffff81c816c000
 R8: 0x00000003169e9807,  R9: 0x0000000000000000, R10: 0xffffff81c8178d00, R11: 0x0000000000000000
R12: 0x000000001fc8bfd0, R13: 0x0000000000000000, R14: 0xffffff81e8ba3588, R15: 0x0000000009a7ec80
RFL: 0x0000000000010246, RIP: 0xffffff7f837537d0,  CS: 0x0000000000000008,  SS: 0x0000000000000010
Fault CR2: 0xffffff81f645e000, Error code: 0x0000000000000000, Fault CPU: 0x8, PL: 0, VF: 0

Backtrace (CPU 8), Frame : Return Address
0xffffff81e8ba3010 : 0xffffff80003aba9d 
0xffffff81e8ba3060 : 0xffffff80004e5bd3 
0xffffff81e8ba30a0 : 0xffffff80004d75fa 
0xffffff81e8ba3110 : 0xffffff8000358ca0 
0xffffff81e8ba3130 : 0xffffff80003ab4b7 
0xffffff81e8ba3250 : 0xffffff80003ab303 
0xffffff81e8ba32c0 : 0xffffff80004d781d 
0xffffff81e8ba3430 : 0xffffff8000358ca0 
0xffffff81e8ba3450 : 0xffffff7f837537d0 
0xffffff81e8ba3570 : 0xffffff7f837514bb 
0xffffff81e8ba3990 : 0xffffff7f837296e5 
0xffffff81e8ba3a00 : 0xffffff7f814dd7c6 
0xffffff81e8ba3a40 : 0xffffff7f814dd67b 
0xffffff81e8ba3a90 : 0xffffff8000a83f68 
0xffffff81e8ba3ae0 : 0xffffff7f814e3c79 
0xffffff81e8ba3b30 : 0xffffff8000a8d3ef 
0xffffff81e8ba3c70 : 0xffffff8000492234 
0xffffff81e8ba3d80 : 0xffffff80003b118d 
0xffffff81e8ba3dd0 : 0xffffff800038bb45 
0xffffff81e8ba3e50 : 0xffffff80003a04fe 
0xffffff81e8ba3ef0 : 0xffffff80004bed4b 
0xffffff81e8ba3fa0 : 0xffffff8000359486 
    Kernel Extensions in backtrace:
        com.apple.iokit.IOGraphicsFamily(530.9)@0xffffff7f814c1000->0xffffff7f8150bfff
            dependency: com.apple.iokit.IOPCIFamily(2.9)@0xffffff7f80c95000
        com.apple.driver.AppleIntelCFLGraphicsFramebuffer(12.0.2)@0xffffff7f83711000->0xffffff7f83912fff
            dependency: com.apple.iokit.IOPCIFamily(2.9)@0xffffff7f80c95000
            dependency: com.apple.iokit.IOACPIFamily(1.4)@0xffffff7f80d10000
            dependency: com.apple.iokit.IOAcceleratorFamily2(400.25)@0xffffff7f82ef0000
            dependency: com.apple.iokit.IOReportFamily(47)@0xffffff7f80ddb000
            dependency: com.apple.AppleGraphicsDeviceControl(3.22.18)@0xffffff7f817d8000
            dependency: com.apple.iokit.IOGraphicsFamily(530.9)@0xffffff7f814c1000
```

此报告表明由于零除错误而触发内核严重错误，我们可以进一步知道它是`AppleIntelFramebufferController::SetupDPTimings(AppleIntelFramebuffer*, AppleIntelDisplayPath*, AppleIntelFramebufferController::CRTCParams*)`在我们反汇编可执行文件后命名的函数内触发的。

对于内置4K显示器的笔记本电脑，我建议使用0x14（HBR2,5.4 Gbps）作为最大链路速率。
对于内置1080p显示器的笔记本电脑，我建议使用0x0A（HBR，2.7 Gbps）。
如果这些值在您的设备上不起作用，您可能需要尝试0x06（RBR，1.62 Gbps）和0x1E（HBR3,8.4 Gbps）。
换句话说，您只需`C1 14`用`C1 06`或`C1 1E`相应地替换最后两个字节。

#### Clover KextsToPatch [二进制]

**支持的操作系统：** macOS Mojave 10.14,10.14.1,10.14.2

```
Info: Set the maximum link rate in DPCD buffer to 0x14 (HBR2) for laptops with 4K display
Name: AppleIntelCFLGraphicsFramebuffer
Find: E8 00 00 00 00 48 83 C3 04 48 83 FB 08 72 D0
Repl: 41 83 BC 24 DC 01 00 00 00 75 04 C6 45 C1 14
Info: Set the maximum link rate in DPCD buffer to 0x0A (HBR) for laptops with 1080p or below display
Name: AppleIntelCFLGraphicsFramebuffer
Find: E8 00 00 00 00 48 83 C3 04 48 83 FB 08 72 D0
Repl: 41 83 BC 24 DC 01 00 00 00 75 04 C6 45 C1 0A
```

#### Clover KextsToPatch

**支持的操作系统：** macOS Mojave 10.14,10.14.1,10.14.2

```
<key>KextsToPatch</key>
<array>
    <dict>
        <key>Comment</key>
        <string>Set the maximum link rate in DPCD buffer to 0x14 (HBR2) for laptops with 4K display (by FireWolf)</string>
        <key>Disabled</key>
        <false/>
        <key>Find</key>
        <data>
        6AAAAABIg8MESIP7CHLQ
        </data>
        <key>InfoPlistPatch</key>
        <false/>
        <key>Name</key>
        <string>AppleIntelCFLGraphicsFramebuffer</string>
        <key>Replace</key>
        <data>
        QYO8JNwBAAAAdQTGRcEU
        </data>
    </dict>
    <dict>
        <key>Comment</key>
        <string>Set the maximum link rate in DPCD buffer to 0x0A (HBR) for laptops with 1080p or below display (by FireWolf)</string>
        <key>Disabled</key>
        <false/>
        <key>Find</key>
        <data>
        6AAAAABIg8MESIP7CHLQ
        </data>
        <key>InfoPlistPatch</key>
        <false/>
        <key>Name</key>
        <string>AppleIntelCFLGraphicsFramebuffer</string>
        <key>Replace</key>
        <data>
        QYO8JNwBAAAAdQTGRcEK
        </data>
    </dict>
</array>
```



## 验证

```bash
/System/Library/Extensions/AppleGraphicsControl.kext/Contents/MacOS/AGDCDiagnose -a | more
```

## 引用链接

https://www.firewolf.science/static/articles/cflfb/
https://www.firewolf.science/2018/11/coffee-lake-intel-uhd-graphics-630-on-macos-mojave-a-nearly-ultimate-solution-to-the-kernel-panic-due-to-division-by-zero-in-the-framebuffer-driver/

## 核显的显示器接口

connector-type的一种定义

- 00 04 00 00 -->DP
- 00 08 00 00 -->HDMI
- 04 00 00 00 -->DVI
- 02 00 00 00 -->LVDS
- 01 00 00 00 -->虚拟显示，可能是远程调用的。

## SMBIOS

- iMac18,1 这个机型是使用核显做显示器输出的，从AGPM的info.plist信息中能看到这种机型为iGPU做了很多的设置。
- iMac18,3/18,2 这个机型独显做显示输出，核显仅参与一些计算任务，这种机型的AGPM的info.plist信息中，iGPU的设置参数做了简化。
- iMacPro1,1 MacProX,X 独显输出，默认不开启核显，而且独显有各种机型的专用显卡硬件ID匹配，没有iGPU的设置参数。
- 其他机型类似，你需要到/System/Library/Extensions/AppleGraphicsPowerManagement.kext/Contents/info.plist里去核对。



