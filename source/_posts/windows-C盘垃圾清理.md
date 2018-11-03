---
title: windows_C盘垃圾清理
date: 2018-11-03 08:46:01
tags: [windows,垃圾清理]
categories: windows
---
windows c盘垃圾清理程序，清理各类软件的缓存文件、日志文件等，长时间未清洗，可以空出10多G的空间出来，具体程序如下，创建一个abc.bat可执行文件（名字随便取），点击运行即可：
<!-- more -->
```
@echo off 
echo 正在清除系统垃圾文件，请稍等...... 
del /f /s /q %systemdrive%\*.tmp 
del /f /s /q %systemdrive%\*._mp 
del /f /s /q %systemdrive%\*.log 
del /f /s /q %systemdrive%\*.gid 
del /f /s /q %systemdrive%\*.chk 
del /f /s /q %systemdrive%\*.old 
del /f /s /q %systemdrive%\recycled\*.* 
del /f /s /q %windir%\*.bak 
del /f /s /q %windir%\prefetch\*.* 
rd /s /q %windir%\temp & md %windir%\temp 
del /f /q %userprofile%\小甜饼s\*.* 
del /f /q %userprofile%\recent\*.* 
del /f /s /q "%userprofile%\Local Settings\Temporary Internet Files\*.*" 
del /f /s /q "%userprofile%\Local Settings\Temp\*.*" 
del /f /s /q "%userprofile%\recent\*.*" 
echo 清除系统LJ完成！ 
echo. & pause  
```
随便放在电脑的一个位置，点击运行即可。