---
title: "[Bat] bat脚本实现文件批量重命名操作"
date: 2021-12-31T16:56:22+08:00
author:     "Rezzo"
categories: [ "Tips" ] 
tags: ["Tutorials", "Bat"]
---
批量删除相同前缀
```
@echo off
title 批量删除前缀名
echo.
echo 本批处理可批量删除前缀名
echo.
echo.&set /p strtemp3= 请输入要删除前缀的文件类型:
echo.&set /p strtemp2= 请输入要删除的前缀字符串:
setlocal enabledelayedexpansion
for /f "delims=" %%i in ('dir /a /b %strtemp2%*.%strtemp3%') do (
rem 只将文件名传给var,而不将扩展名传递
rem set var=%%~ni
rem 将文件名传给var
set var=%%i
rem 当使用变量延迟时，不再使用%%括，而用!!括，否则变量延迟无效
set var=!var:%strtemp2%=!
echo %%i !var!
rem 重命名
ren "%%i" "!var!"
rem 输出一个空行
)
echo.
echo OK了！
echo.
pause
```
批量添加前缀
```
@echo off
for %%i in (*.png)  do ren "%%i" perfix"%%i"
```
