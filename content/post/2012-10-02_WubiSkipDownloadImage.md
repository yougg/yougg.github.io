---
Categories: []
Description: ""
Tags: []
title: "Wubi安装Ubuntu, 通过调试日志发现如何跳过自动下载镜像文件"
date: 2012-10-02T00:00:00+08:00
draft: false
---

以下是执行一次wubi安装ubuntu的调试日志:

> 10-02 16:23 INFO   root: === wubi 12.10 rev270 ===  
10-02 16:23 DEBUG  root: Logfile is c:\users\用户名\appdata\local\temp\wubi-12.10-rev270.log  
10-02 16:23 DEBUG  root: sys.argv = ['main.pyo', '--exefile="Z:\\wubi.exe"']  
10-02 16:23 DEBUG  CommonBackend: data_dir=C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\data  
10-02 16:23 DEBUG  WindowsBackend: 7z=C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\bin\7z.exe  
10-02 16:23 DEBUG  WindowsBackend: startup_folder=C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup  
10-02 16:23 DEBUG  CommonBackend: Fetching basic info...  
10-02 16:23 DEBUG  CommonBackend: original_exe=Z:\wubi.exe  
10-02 16:23 DEBUG  CommonBackend: platform=win32  
10-02 16:23 DEBUG  CommonBackend: osname=nt  
10-02 16:23 DEBUG  WindowsBackend: arch=amd64  
10-02 16:23 DEBUG  CommonBackend: Parsing isolist=C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\data\isolist.ini  
10-02 16:23 DEBUG  CommonBackend:   Adding distro Edubuntu-i386  
10-02 16:23 DEBUG  CommonBackend:   Adding distro Kubuntu-amd64  
10-02 16:23 DEBUG  CommonBackend:   Adding distro Mythbuntu-i386  
10-02 16:23 DEBUG  CommonBackend:   Adding distro Edubuntu-amd64  
10-02 16:23 DEBUG  CommonBackend:   Adding distro Ubuntu-amd64  
10-02 16:23 DEBUG  CommonBackend:   Adding distro Lubuntu-i386  
10-02 16:23 DEBUG  CommonBackend:   Adding distro Ubuntu-i386  
10-02 16:23 DEBUG  CommonBackend:   Adding distro Mythbuntu-amd64  
10-02 16:23 DEBUG  CommonBackend:   Adding distro Kubuntu-i386  
10-02 16:23 DEBUG  CommonBackend:   Adding distro Lubuntu-amd64  
10-02 16:23 DEBUG  WindowsBackend: Fetching host info...  
10-02 16:23 DEBUG  WindowsBackend: registry_key=Software\Microsoft\Windows\CurrentVersion\Uninstall\Wubi  
10-02 16:23 DEBUG  WindowsBackend: windows version=vista  
10-02 16:23 DEBUG  WindowsBackend: windows_version2=Windows 7 Professional  
10-02 16:23 DEBUG  WindowsBackend: windows_sp=None  
10-02 16:23 DEBUG  WindowsBackend: windows_build=7600  
10-02 16:23 DEBUG  WindowsBackend: gmt=8  
10-02 16:23 DEBUG  WindowsBackend: country=CN  
10-02 16:23 DEBUG  WindowsBackend: timezone=Asia/Shanghai  
10-02 16:23 DEBUG  WindowsBackend: windows_username=用户名  
10-02 16:23 DEBUG  WindowsBackend: user_full_name=用户名  
10-02 16:23 DEBUG  WindowsBackend: user_directory=C:\Users\用户名  
10-02 16:23 DEBUG  WindowsBackend: windows_language_code=1028  
10-02 16:23 DEBUG  WindowsBackend: windows_language=Chinese (Traditional)  
10-02 16:23 DEBUG  WindowsBackend: processor_name=Intel(R) Core(TM)2 Duo CPU     T7700  @ 2.40GHz  
10-02 16:23 DEBUG  WindowsBackend: bootloader=vista  
10-02 16:23 DEBUG  WindowsBackend: system_drive=Drive(C: hd 25134.0 mb free ntfs)  
10-02 16:23 DEBUG  WindowsBackend: drive=Drive(C: hd 25134.0 mb free ntfs)  
10-02 16:23 DEBUG  WindowsBackend: drive=Drive(D: hd 25134.0 mb free ntfs)  
10-02 16:23 DEBUG  WindowsBackend: drive=Drive(E: hd 15735.46875 mb free ntfs)  
10-02 16:23 DEBUG  WindowsBackend: drive=Drive(F: hd 20273.5546875 mb free ntfs)  
10-02 16:23 DEBUG  WindowsBackend: drive=Drive(Z: cd 0.0 mb free cdfs)  
10-02 16:23 DEBUG  WindowsBackend: uninstaller_path=None  
10-02 16:23 DEBUG  WindowsBackend: previous_target_dir=None  
10-02 16:23 DEBUG  WindowsBackend: previous_distro_name=None  
10-02 16:23 DEBUG  WindowsBackend: keyboard_id=134481924  
10-02 16:23 DEBUG  WindowsBackend: keyboard_layout=us  
10-02 16:23 DEBUG  WindowsBackend: keyboard_variant=  
10-02 16:23 DEBUG  WindowsBackend: total_memory_mb=2030.296875  
10-02 16:23 DEBUG  CommonBackend: Searching ISOs on USB devices  
10-02 16:23 DEBUG  CommonBackend: Searching for local CDs  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Kubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Kubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Mythbuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Mythbuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Edubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Edubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Lubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Lubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Kubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Kubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Mythbuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Mythbuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Edubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Edubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Lubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Lubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Kubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Kubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Mythbuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Mythbuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Edubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Edubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Lubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Lubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Kubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Kubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Mythbuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Mythbuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Edubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Edubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Lubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Lubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Kubuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Kubuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Mythbuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Mythbuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Edubuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Edubuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Lubuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Lubuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 INFO   root: Running the installer...  
10-02 16:23 INFO   WinuiPage: appname=wubi, localedir=C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\translations, languages=['zh_CN', 'zh']  
10-02 16:23 INFO   WinuiPage: appname=wubi, localedir=C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\translations, languages=['zh_CN', 'zh']  
10-02 16:23 DEBUG  WinuiInstallationPage: target_drive=D:, installation_size=15000MB, distro_name=Ubuntu, language=en_US, locale=en_US.UTF-8, username=用户名  
10-02 16:23 INFO   root: Received settings  
10-02 16:23 DEBUG  CommonBackend: Searching for local CD  
10-02 16:23 DEBUG  Distro:   checking whether C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether D:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain D:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether E:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain E:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether F:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro:     does not contain F:\casper\filesystem.squashfs  
10-02 16:23 DEBUG  Distro:   checking whether Z:\ is a valid Ubuntu CD  
10-02 16:23 DEBUG  Distro: could not get info None  
10-02 16:23 DEBUG  CommonBackend: Searching for local ISO  
10-02 16:23 INFO   WinuiPage: appname=wubi, localedir=C:\Users\用户名\AppData\Local\Temp\pylF3F0.tmp\translations, languages=['en_US', 'en']  
10-02 16:23 DEBUG  TaskList: # Running tasklist...  
10-02 16:23 DEBUG  TaskList: ## Running select_target_dir...  
10-02 16:23 INFO   WindowsBackend: Installing into D:\ubuntu  
10-02 16:23 DEBUG  TaskList: ## Finished select_target_dir  
10-02 16:23 DEBUG  TaskList: ## Running create_dir_structure...  
10-02 16:23 DEBUG  CommonBackend: Creating dir D:\ubuntu  
10-02 16:23 DEBUG  CommonBackend: Creating dir D:\ubuntu\disks  
10-02 16:23 DEBUG  CommonBackend: Creating dir D:\ubuntu\install  
10-02 16:23 DEBUG  CommonBackend: Creating dir D:\ubuntu\install\boot  
10-02 16:23 DEBUG  CommonBackend: Creating dir D:\ubuntu\disks\boot  
10-02 16:23 DEBUG  CommonBackend: Creating dir D:\ubuntu\disks\boot\grub  
10-02 16:23 DEBUG  CommonBackend: Creating dir D:\ubuntu\install\boot\grub  
10-02 16:23 DEBUG  TaskList: ## Finished create_dir_structure  
10-02 16:23 DEBUG  TaskList: ## Running create_uninstaller...  
10-02 16:23 DEBUG  WindowsBackend: Copying uninstaller Z:\wubi.exe -> D:\ubuntu\uninstall-wubi.exe  
10-02 16:23 DEBUG  registry: Setting registry key -2147483646 Software\Microsoft\Windows\CurrentVersion\Uninstall\Wubi UninstallString D:\ubuntu\uninstall-wubi.exe  
10-02 16:23 DEBUG  registry: Setting registry key -2147483646 Software\Microsoft\Windows\CurrentVersion\Uninstall\Wubi InstallationDir D:\ubuntu  
10-02 16:23 DEBUG  registry: Setting registry key -2147483646 Software\Microsoft\Windows\CurrentVersion\Uninstall\Wubi DisplayName Ubuntu  
10-02 16:23 DEBUG  registry: Setting registry key -2147483646 Software\Microsoft\Windows\CurrentVersion\Uninstall\Wubi DisplayIcon D:\ubuntu\Ubuntu.ico  
10-02 16:23 DEBUG  registry: Setting registry key -2147483646 Software\Microsoft\Windows\CurrentVersion\Uninstall\Wubi DisplayVersion 12.10-rev270  
10-02 16:23 DEBUG  registry: Setting registry key -2147483646 Software\Microsoft\Windows\CurrentVersion\Uninstall\Wubi Publisher Ubuntu  
10-02 16:23 DEBUG  registry: Setting registry key -2147483646 Software\Microsoft\Windows\CurrentVersion\Uninstall\Wubi URLInfoAbout http://www.ubuntu.com  
10-02 16:23 DEBUG  registry: Setting registry key -2147483646 Software\Microsoft\Windows\CurrentVersion\Uninstall\Wubi HelpLink http://www.ubuntu.com/support  
10-02 16:23 DEBUG  TaskList: ## Finished create_uninstaller  
10-02 16:23 DEBUG  TaskList: ## Running create_preseed_diskimage...  
10-02 16:23 DEBUG  TaskList: ## Finished create_preseed_diskimage  
10-02 16:23 DEBUG  TaskList: ## Running get_diskimage...  
10-02 16:23 DEBUG  TaskList: New task download  
10-02 16:23 DEBUG  TaskList: ### Running download...  
10-02 16:23 DEBUG  downloader: downloading http://releases.ubuntu.com/12.10/ubuntu-12.10-wubi-amd64.tar.xz > D:\ubuntu\disks\ubuntu-12.10-wubi-amd64.tar.xz  
10-02 16:23 DEBUG  TaskList: # Finished tasklist

通过以上日志发现:

wubi搜索镜像文件的顺序如下:

1. Live USB (就是U盘镜像了)
2. 本地光驱中的镜像 (使用物理光驱挂载CD光盘)
3. 本地硬盘分区镜像 (使用虚拟光驱挂载下载的ISO镜像文件)
4. 本地ISO镜像文件 (和wubi.exe放在同一个目录)
5. 联网自动下载

只要搜索到了前4步中的任何一个可用的镜像文件,wubi就会将其复制到安装目录进行安装了.  
这样就不用漫漫等待wubi自身下载浪费的大把时间了.
