+++
date = '2026-02-18T10:58:36+08:00'
draft = false
slug = 8799aafa  # 随机生成8位字符串
type = posts
title = 'Scoop入门'
collection = "scoop"
categories = "系统"
+++

# scoop 


## 放在最上面： 新电脑初始化步骤
```
# 1. 设置安装路径到 D 盘
$env:SCOOP = 'D:\Scoop'
[Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')

# 2. 安装 Scoop
irm get.scoop.sh | iex

# 3. 配置代理和加速
scoop config proxy 127.0.0.1:7890
scoop install aria2
scoop config aria2-enabled true

# 4. 添加常用源
scoop bucket add extras
scoop bucket add java

# 5. 安装开发全家桶
scoop install git python nvm openjdk17 vscode

# 6.安装node
nvm install 20.20.0
nvm use 20.20.0
```


## 什么是scoop
是windows 的一个命令行安装包管理器，使用命令行来安装软件，类似于linux的apt

核心思路：
	将对应软件的压缩包解压到 ~/scoop 目录 ，使用命令行快捷方式来将可执行文件暴露给PATH

与传统的 setup.exe 的区别（scoop的特性）：
- scoop安装的是绿色版，然后解压到对应目录，减少系统污染
- 不需要弹出以管理员权限运行弹出
- 不用找各个软件的下载地址，直接一行命令就能安装
- 不用配置环境变量，scoop会自动配置
- 会自动安装软件需要的依赖

适用：
- 开发人员 快速安装开发环境
- 对系统洁癖的用户
## 安装
官方文档：[ScoopInstaller/Scoop：一款适用于 Windows 的命令行安装程序。](https://github.com/ScoopInstaller/Scoop)

scoop会默认安装在 c盘用户目录下的scoop目录中，我们需要将其改到d盘再安装，节省c盘空间
步骤：
1. 设置当前会话变量
	`$env:SCOOP = 'D:\Scoop'
2. 允许命令行执行脚本（windows10 默认阻止命令行执行脚本，需要开启）
	打开终端执行：`Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`
3. 执行获取scoop的脚本，并执行
	执行 `irm get.scoop.sh | iex` 
4. 验证是否安装成功
	`scoop --version`


安装完成后可以打开 D:\Scoop查看目录结构：
	![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260218083841314.png)

## 使用示例1：安装aria2
aria2是一个命令行下载工具，可以实现多连接下载

如何使用scoop安装
1.  搜索安装包
	`scoop search aria2`
	![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260218084311991.png)
2. 安装软件
	`scoop install aria2`


## 常用命令
搜索软件:
`scoop search <name>`

安装软件：
`scoop install <name>`        或者       `scoop i <name>`

卸载软件：
`scoop uninstall <name>`       或者       `scoop rm <name>`

查看已安装的软件：
`scoop list`  或者 `scoop ls`

更新软件：
`scoop update` 更新scoop自身                或者      `scoop u `
`scoop update *`  更新所有已安装软件         或者      `scoop u *`


## 软件仓库
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260218083841314.png)
上图中的buckets就是软件仓库
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260218085034879.png)

你目前应该只有 main 一个仓库。extras 和java 都是需要额外添加的

也就是说scoop官方只提供了main仓库，我们可以添加其他的软件仓库

### scoop常用的软件仓库：
- main: 默认源，命令行工具为主。
- extras: 必加，包含常用 GUI 软件（Chrome, VSCode, 微信等）。
- java: 各种版本的 JDK。
- versions: 软件的历史旧版本。
- nirsoft: NirSoft 工具集。

### 管理仓库命令
添加仓库:
	`scoop bucket add <name>`
列出已添加的
	`scoop bnucket list`
查看官方支持的仓库
	`scoop bucket known`
删除仓库
	`scoop bucket remove <name>`

安装仓库的方式： 就是使用git clone


## 使用示例2：安装java8
1. 添加java仓库
	`scoop bucket add java`
2. 搜索
	`scoop search jdk`
	你会看到有  openjdk8-redhat，temurin8-jdk 等等一堆
	我们可以选择一个安装就好： 比如temurin8-jdk，Eclipse Adoptium 维护的版本，稳定性好且免费。
	或者openjdk8-redhat 标准jdk
3. 安装
	`scoop install openjdk8-redhat`
4. 验证是否安装成功
	`java -version`

不需要配置环境变量吗？
不需要

### 多版本java切换
比如你又安装了一个openjdk21

查看一下所有安装的jdk
`scoop list | findstr jdk`
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260218090940398.png)

`java -version`
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260218091129511.png)

此时你的java环境是 jdk21， 你想要切换到jdk8 如何切换

`scoop reset temurin8`       这个命令的意思是重置软件的链接（Shim），也就是path指向
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260218091147429.png)

就切换到了jdk8