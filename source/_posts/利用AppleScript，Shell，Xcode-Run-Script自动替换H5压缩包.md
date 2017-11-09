---
title: 利用AppleScript，Shell，Xcode_Run_Script自动替换H5压缩包
date: 2017-11-09 17:27:23
tags: Xcode小技巧
---
# 准备
公司的App是Hybird架构的App，发版本前需要将H5压缩文件放在工程指定路径下。并且在平时开发测试过程中，经常需要替换H5压缩文件，感觉很麻烦，便想到了将整个过程自动化。开始之前有以下几个问题:

- 不同开发人员项目工程路径不统一导致H5压缩文件存放路径不一致

> 将脚本文件与H5压缩文件放在统一目录下，获取脚本文件路径就是H5压缩文件的存放路径

- 如何获取新的H5压缩文件

> 设定新H5压缩文件的存放策略，是获取路径固定

- 通过何种方式促发脚本运行

> Xcode 自带的Run Script支持在`Build`的时候运行脚本

# 实施
## AppleScript 
本着高校快速解决问题的原则，我直接选用了macOS自带的『脚本编辑器』应用，利用`AppleScript`书写脚本。

> 设定新的H5压缩文件的存放策略是名称为`h5`，并且置于`Desktop`

```
tell application "Finder"
    tell application "Xcode"
        display dialog "确定移动桌面上名为h5的文件吗?" buttons {"Yes", "No"}
    end tell
    if button returned of result is "Yes" then
        try
            set destination to container of (path to me) as alias
            set origin to the path to the desktop folder as alias
            move file "h5.zip" of origin to destination with replacing
            delete file "H5Resource.zip" of destination
            set the name of file "h5.zip" of destination to "H5Resource.zip"
        on error
            tell application "Xcode"
                display dialog "错误:请确保桌面上有名为'h5.zip'的压缩文件" buttons {"Yes"}
            end tell
        end try
    end if
end tell
```

在程序中做好异常处理以及相关提示，然后再导出(H5Replacer.scpt)存放到工程H5压缩文件目录下。
这时已经可以半自动的替换了，就是在存放好新的h5压缩文件以后，手动触发脚本。

## Shell & Run Script
之后是书写`Shell`脚本，实现在`Build`或者`Run`的时候自动触发脚本。

```
h5Path=/Users/${USER}/Desktop/h5.zip
if [ -e $h5Path ]
then
    ascript=${SRCROOT}/${TARGET_NAME}/Resources/H5 Resource/H5Replacer.scpt
    if [ -x $ascript ]
    then
        osascript ${SRCROOT}/${TARGET_NAME}/Resources/H5\ Resource/H5Replacer.scpt
    else
        osascript -e 'tell app "Xcode" to display dialog "Xcode:请确保H5Replacer.scpt文件位置正确" buttons {"YES"}'
    fi
else
    echo [script message]:未执行替换h5的脚本
fi
```

> WARN：注意H5 Resource文件夹中存在空格时的处理

这段代码会去检查桌面上是否存在名为h5的压缩文件，如果存在便去执行`H5Replacer.scpt`脚本，同时做好异常处理。
这样我们就能自动替换H5压缩文件了。

>在书写过程中也许需要看到一些打印信息，可以选中最近一次编译或运行，同过过滤条件来查看`Run Script`打印的信息
>![](http://oy9at3amc.bkt.clouddn.com/A239F2F4-273B-400C-B354-6E6E77D3CE45.png)

# 写在最后
在这过程中发现有意思的在`Run Script`中，如果遇到需要用户选择时，Xcode会暂停编译过程，等待用户选择。
![](http://oy9at3amc.bkt.clouddn.com/62387317.png)