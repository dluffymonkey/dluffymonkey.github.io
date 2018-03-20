---
title: iOS自动化打包脚本
date: 2017-12-24
tags: iOS
categories: iOS Tips
---

#### iOS自动化打包脚本(shell)


**本脚本主要作用为代替人工打包app，导出ipa包并安装的过程，如果是AppStore方式，会自动上传AppStore，不需要手动管理。 如需使用自动安装ipa功能，需要进行一些额外的环境配置。**

##### 打开autoarchive.sh脚本

<img src="http://img.blog.csdn.net/20170401102400273?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="30%" />


主要配置如上图

**projectName为当前待打包工程的工名**



**下面有四种类型，分别代表打包使用的证书类型，此类型可添加，删除等。**

> + development 为开发证书打包
> + enterprise为企业证书打包
> + appstore为上传AppStore打包
> + adhoc为发布证书打包，发布证书一般用于安装在指定的设备上

 <!-- more -->

_**四种类型的名称需要与当前工程的Scheme名称后缀一致。**_

**如何为当前工程添加多个Scheme呢，步骤如下**

***为工程添加多个target与Scheme***

+ 使用Xcode打开当前工程，点击工程，选择targets中的任意一个


+ 右键-> Duplicate,选择Duplicate Only，复制当前的target,并将复制好的target改名为QSArchiveTest_Enterprise,如图所示
  <img src="http://img.blog.csdn.net/20170401102441596?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="30%" />
  <img src="http://img.blog.csdn.net/20170401102503243?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="30%" />

+ 修改QSArchiveTestcopy-Info.plist名称为QSArchiveTest _Enterprise-Info.plist，点击工程，选择target QSArchiveTest_Enterprise,点击Build Settings ,搜索plist，将如图所示的QSArchiveTest copy-Info.plist名称修改为QSArchiveTest_Enterprise-Info.plist ，然后点击info,如果成功，则如图，若不成功，可能是复制文档中名称错误，请从工程文件中复制文件名。
    <center>
    <img src="http://img.blog.csdn.net/20170401102612675?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="30%" />
        <img src="http://img.blog.csdn.net/20170401102633722?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="30%" />
    </center>

+ 为当前target添加Scheme。点击当前Scheme,选择Manage Schemes, 删除QSArchiveTest copy，然后选择新建，选择当前已新建好的target，点击ok
    <center>
    <img src="http://img.blog.csdn.net/20170401102805794?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="30%" />
    </center>
    **以上步骤成功新建了一个名为QSArchiveTest_Enterprise的Scheme，此时，当前工程可以运行在QSArchiveTest_Enterprise Scheme上，接下来，我们可以按照以上步骤新建QSArchiveTest_Develoment ，QSArchiveTest_AppStore, QSArchiveTest_AdHoc等Schemes**
    <center>
    <img src="http://img.blog.csdn.net/20170401103014501?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="30%" height="30%" />
    </center>


**结果如图所示**
    <center>
    <img src="http://img.blog.csdn.net/20170401103110815?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="50%" />
    </center>


#### 为各个target配置不同的证书及mobileprovision，不同的配置将生成不同的product，也就是不同的app。下面以企业证书为例####

+ 点击工程，选择target->QSArchiveTest_Enterprise,点击general,将bundle identifier修改为com.xxxx.xxxx

+ 点击build settings,向下滑动到签名栏，先选择provisioning profile，然后选择证书，如图所示，配置完成后点击general，如果出现如下图所示，则配置成功。如果有警告或者错误提示，则检查上面的步骤是否正确。**
    <img src="http://img.blog.csdn.net/20170401103230443?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="50%" />

    <img src="http://img.blog.csdn.net/20170401103304912?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="50%" />


**重复以上步骤，配置target， QSArchiveTest_Development，QSArchiveTest_AppStore，QSArchiveTest_AdHoc，分别选择对应的证书及mobileprovision。**

### 添加导出设置plist文件###

+ 在当前项目的根目录下新建文件夹，名称为autobuild，进入autobuild，使用Xcode新建plist文件，名称为EnterpriseExportOptions.plist,将文件保存到autobuild文件夹中
    <img src="http://img.blog.csdn.net/20170401103433835?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="50%" />

+ 为EnterpriseExportOptions.plist添加键值对，使用Xcode打开刚才新建的plist文件，为其添加如下键值对
    ![这里写图片描述](http://img.blog.csdn.net/20170401103553134?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**method为必选项，有四种选项，分别为**[app-store](), [ad-hoc](), [enterprise](), [development]()。

*各个plist的键值对如下*

**AppStoreExportOptions.plist：**

>  method＝**app-store**，uploadBitcode＝YES，uploadSymbols＝YES

**EnterpriseExportOptions.plist：**

> method＝**enterprise**，compileBitcode＝NO

**DevelopmentExportOptions.plist：**

> method＝**development**，compileBitcode＝NO

**DevelopmentExportOptions.plist：**

> method＝**ad-hoc**，compileBitcode＝NO

**获取不同的target对应的证书名称及mobileprovision的uuid，以下以企业证书为例**

+ 点击工程，选择target-> QSArchiveTest_Enterprise，点击build settings,滑动到Signing，点击Provisioning Profile(Deprecated)栏，选择other，拷贝uuid，替换autoarchive.sh脚本中的enterpriseProvisioningProfile变量值，如图所示
  ![这里写图片描述](http://img.blog.csdn.net/20170401103816605?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

  ![这里写图片描述](http://img.blog.csdn.net/20170401103901182?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


+ 点击Code Signing Identity,选择other，拷贝证书名，替换autoarchive.sh中的** enterpriseCodeSignIdentity变量值，如图所示
    ![这里写图片描述](http://img.blog.csdn.net/20170401103935466?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

    ![这里写图片描述](http://img.blog.csdn.net/20170401104035046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**重复以上步骤，分别获取证书名称及provisionfile的 uuid，替换autoarchive.sh中的CodeSignIdentity，ProvisioningProfile对应变量的值，结果如图所示。此处未有adhoc证书，因此置为空**
    <center>
    <img src="http://img.blog.csdn.net/20170401104112777?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="50%" />
    </center>

**如需使用上传至AppStore功能，需在脚本中设置Apple ID如图，将自己的Apple ID以及密码替换即可**
    <center>
    <img src="http://img.blog.csdn.net/20170401104147808?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="50%" />
    </center>

**至此，已完成了自动化脚本执行所需要的配置，可以开始使用脚本进行打包了。**

##### 使用autoarchive.sh进行打包，以企业证书包为例###

**打开命令行，cd到当前工程的根目录，使用如下命令执行脚本**

`./autoarchive.sh –t Enterprise`

**若有错误提示，使用如下命令解决**

`chmod +x autoarchive.sh`

**再次执行脚本，等待打包过程结束，如果当前连接了设备，请先将设备上的应用删除，脚本打包完成后将会自动将ipa包安装到设备中（如需成功安装ipa到设备，需要查看下文-脚本详解，按照其中的步骤安装相应的工具）。成功如下图所示，如失败，请查看错误提示，并参照错误提示检查前面的步骤。打包完成后会自动打开当前ipa包所在目录，如未打开，请拷贝exportPath路径，打开Finder，使用快捷键 Command + shilf + g，打开ipa包路径。**
    <img src="http://img.blog.csdn.net/20170401104227309?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="50%" />
    <img src="http://img.blog.csdn.net/20170401104246184?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" width="50%" height="50%" />

#### 脚本详解

**本脚本包括以下几个方法**

+ clean方法，作用为clean工程，日志将会输出到log.txt中，使用xcodebuild命令执行，关于xcodebuild命令的详细情况请使用xcodebuild –help了解，如无法使用xcodebuild，请检查mac使用安装了Xcode，如已安装，请检查是否设置Xcode为当前命令行工具，检查方法如图
    ![这里写图片描述](http://img.blog.csdn.net/20170401104337145?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

    ![这里写图片描述](http://img.blog.csdn.net/20170401104404137?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

+ archive方法，archive方法主要为打包QSArchiveTest.xcarchive所用
    ![这里写图片描述](http://img.blog.csdn.net/20170401104440756?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

+ export方法，此方法为导出ipa包，导出路径自定义

  ![这里写图片描述](http://img.blog.csdn.net/20170401104506741?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

+ **install方法，该方法生效需要安装**ideviceinstaller，libimobiledevice两个工具。两个工具可以使用homebrew进行安装。这两个工具用于安装ipa或者管理iOS设备应用，查看当前连接设备的信息等。****
    ![这里写图片描述](http://img.blog.csdn.net/20170401104534710?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#####安装homebrew

​ **homebrew为macOS不可或缺的套件管理器，$brew install wget**

**安装方式如下**

**打开终端，拷贝以下脚本,回车，等待安装结束**

`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

##### 安装ideviceinstaller以及libimobiledevice。

安装如下命令逐个执行，前两条用于卸载，如果已安装，但是改工具使用不正常，即可使用命令卸载

> brew uninstall ideviceinstaller
>
> brew uninstall libimobiledevice
>
> brew install --HEAD libimobiledevice
>
> brew link --overwrite libimobiledevice
>
> brew install ideviceinstaller
>
> brew link --overwrite ideviceinstaller

**安装完成后使用，查看当前是否有iOS设备连接**

`idevice_id –l`

**查看帮助信息**

`ideviceinstaler –h `

**将ipa包安装至iOS设备**

`ideviceinstaller -i ipaPath`

**ipaPath为需要安装的ipa的路径**

**如果出现以下错误**

`couldnot connect to lockdownd. exiting.`

可以使用指令解决

`sudochmod -R 777 /var/db/lockdown/`

**或者永久的解决办法为重新进行ideviceinstaller安装过程**

**ideviceinstaller工具的功能还有很多，此处不再详细解释，可自行探索**

+ upload方法，该方法用于上传ipa至AppStore，只有在AppStore模式下才会执行。Upload方法用到了Xcode自带的工具Application Loader altool，与手动上传方法一致。altool位于Application Loader中，三个参数ipaPath,appleId,applepassword,ipaPath为ipa包导出的路径，applied为开发者帐号，applepassword为开发者帐号的密码
    ![这里写图片描述](http://img.blog.csdn.net/20170401104608341?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQ1ODgwOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)







方法二

本文最终实现的是使用脚本打 Ad-hoc 包，并发布测试，当然稍微修改一下脚本参数就可以打其他类型的 ipa 包了。另外该脚本还实现了将生成的 ipa 包上传至蒲公英进行测试分发。文中内容包括:

1. xcodebuild 简介
2. 使用xcodebuild和xcrun打包签名
3. 将打包过程脚本化

## xcodebuild 简介

`xcodebuild` 是苹果提供的打包项目或者工程的命令，了解该命令最好的方式就是使用 `man xcodebuild` 查看其 man page. 尽管是英文，一定要老老实实的读一遍就好了。

> DESCRIPTION
>
> xcodebuild builds one or more targets contained in an Xcode project, or builds a scheme contained in an Xcode workspace or Xcode project.

Usage

> To build an Xcode project, run xcodebuild from the directory containing your project (i.e. the directory containing the name.xcodeproj package). If you have multiple projects in the this directory you will need to use -project to indicate which project should be built.  By default, xcodebuild builds the first target listed in the project, with the default build configuration. The order of the targets is a property of the project and is the same for all users of the project.
>
> To build an Xcode workspace, you must pass both the -workspace and -scheme options to define the build.  The parameters of the scheme will control which targets are built and how they are built, although you may pass other options to xcodebuild to override some parameters of the scheme.
>
> There are also several options that display info about the installed version of Xcode or about projects or workspaces in the local directory, but which do not initiate an action.  These include -list, -showBuildSettings, -showsdks, -usage, and -version.

总结一下:

1. 需要在包含 name.xcodeproj 的目录下执行 `xcodebuild` 命令，且如果该目录下有多个 projects，那么需要使用 `-project` 指定需要 build 的项目。
2. 在不指定 build 的 target 的时候，默认情况下会 build project 下的第一个 target
3. 当 build workspace 时，需要同时指定 `-workspace` 和 `-scheme` 参数，scheme 参数控制了哪些 targets 会被 build 以及以怎样的方式 build。
4. 有一些诸如 `-list`, `-showBuildSettings`, `-showsdks` 的参数可以查看项目或者工程的信息，不会对 build action 造成任何影响，放心使用。

那么，`xcodebuild` 究竟如何使用呢？ 继续看文档:

> NAME
>
> xcodebuild -- build Xcode projects and workspaces
>
> SYNOPSIS
>
> 1. xcodebuild [-project name.xcodeproj] [[-target targetname] ... | -alltargets] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [action ...] [buildsetting=value ...] [-userdefault=value ...]
> 2. xcodebuild [-project name.xcodeproj] -scheme schemename [[-destination destinationspecifier] ...] [-destination-timeout value] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [action ...] [buildsetting=value ...] [-userdefault=value ...]
> 3. xcodebuild -workspace name.xcworkspace -scheme schemename [[-destination destinationspecifier] ...] [-destination-timeout value] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [action ...] [buildsetting=value ...] [-userdefault=value ...]
> 4. xcodebuild -version [-sdk [sdkfullpath | sdkname]] [infoitem]
> 5. xcodebuild -showsdks
> 6. xcodebuild -showBuildSettings [-project name.xcodeproj | [-workspace name.xcworkspace -scheme schemename]]
> 7. xcodebuild -list [-project name.xcodeproj | -workspace name.xcworkspace]
> 8. xcodebuild -exportArchive -archivePath xcarchivepath -exportPath destinationpath -exportOptionsPlist path
> 9. xcodebuild -exportLocalizations -project name.xcodeproj -localizationPath path [[-exportLanguage language] ...]
> 10. xcodebuild -importLocalizations -project name.xcodeproj -localizationPath path

挑几个我常用的形式介绍一下，较长的使用方式以序列号代替:

- `xcodebuild -showsdks`: 列出 Xcode 所有可用的 SDKs
- `xcodebuild -showBuildSettings`: 上述序号6的使用方式，查看当前工程 build setting 的配置参数，Xcode 详细的 build setting 参数参考官方文档 [Xcode Build Setting Reference](https://link.jianshu.com?t=https://developer.apple.com/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/)， 已有的配置参数可以在终端中以 `buildsetting=value` 的形式进行覆盖重新设置.
- `xcodebuild -list`: 上述序号7的使用方式，查看 project 中的 targets 和 configurations，或者 workspace 中 schemes, 输出如下:

```objective-c
Information about project "NavTabBar":
    Targets:
        NavTabBar
        NavTabBarTests
        NavTabBarUITests

    Build Configurations:
        Debug
        Release
        Ad-hoc

    If no build configuration is specified and -scheme is not passed then "Release" is used.

    Schemes:
        NavTabBar

```

- `xcodebuild [-project name.xcodeproj] [[-target targetname] ... | -alltargets] build`: 上述序号1的使用方式，会 build 指定 project，其中 `-target` 和 `-configuration` 参数可以使用 `xcodebuild -list` 获得，`-sdk` 参数可由 `xcodebuild -showsdks` 获得，`[buildsetting=value ...]` 用来覆盖工程中已有的配置。可覆盖的参数参考官方文档 [Xcode Build Setting Reference](https://link.jianshu.com?t=https://developer.apple.com/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/), `action...` 的可用选项如下, 打包的话当然用 build，这也是默认选项。
  - build
    Build the target in the build root (SYMROOT).  This is the default action, and is used if no action is given.
  - analyze
    Build and analyze a target or scheme from the build root (SYMROOT).  This requires specifying a scheme.
  - archive
    Archive a scheme from the build root (SYMROOT).  This requires specifying a scheme.
  - test
    Test a scheme from the build root (SYMROOT).  This requires specifying a scheme and optionally a destination.
  - installsrc
    Copy the source of the project to the source root (SRCROOT).
  - install
    Build the target and install it into the target's installation directory in the distribution root (DSTROOT).
  - clean
    Remove build products and intermediate files from the build root (SYMROOT).


- `xcodebuild -workspace name.xcworkspace -scheme schemename build`: 上述序号3的使用方式，build 指定 workspace，当我们使用 CocoaPods 来管理第三方库时，会生成 xcworkspace 文件，这样就会用到这种打包方式.

## 使用xcodebuild和xcrun打包签名

开始之前，可以新建一个测试工程 TestImg 来练习打包，在使用终端命令打包之前，请确认该工程也可以直接使用 Xcode 真机调试成功。

然后，打开终端，进入包含 TestImg.xcodeproj 的目录下，运行以下命令:

`xcodebuild -project TestImg.xcodeproj -target TestImg -configuration Release`

如果 build 成功，会看到 `** BUILD SUCCEEDED **` 字样，且在终端会打印出这次 build 的签名信息，如下:

> Signing Identity:     "iPhone Developer: xxx(59xxxxxx)"
> Provisioning Profile: "iOS Team Provisioning Profile: *"

且在该目录下会多出一个 `build` 目录，该目录下有 `Release-iphoneos` 和 `TestImg.build` 文件，根据我们 build `-configuration` 配置的参数不同，`Release-iphoneos` 的文件名会不同。

在 `Release-iphoneos` 文件夹下，有我们需要的`TestImg.app`文件，但是要安装到真机上，我们需要将该文件导出为ipa文件，这里使用 xcrun 命令。

`xcrun -sdk iphoneos -v PackageApplication ./build/Release-iphoneos/TestImg.app -o ~/Desktop/TestImg.ipa`

这里又冒出一个 `PackageApplication`, 我刚开始也不知道这是个什么玩意儿，万能的google告诉我，这是 Xcode 包里自带的工具，使用 `xcrun -sdk iphoneos -v PackageApplication -help` 查看帮助信息.

> Usage:
> PackageApplication [-s signature] application [-o output_directory] [-verbose] [-plugin plugin] || -man || -help
>
> Options:
>
> `[-s signature]`: certificate name to resign application before packaging
> `[-o output_directory]`: specify output filename
> `[-plugin plugin]`: specify an optional plugin
> `-help`: brief help message
> `-man`: full documentation
> `-v[erbose]`: provide details during operation

如果执行成功，则会在你的桌面生成 TestImg.ipa 文件，这样就可以发布测试了。如果你遇到以下警告信息:

> Warning: --resource-rules has been deprecated in Mac OS X >= 10.10!    ResourceRules.plist: cannot read resources

请参考 stackoverflow [这个回答](https://link.jianshu.com?t=http://stackoverflow.com/questions/32504355/error-itms-90339-this-bundle-is-invalid-the-info-plist-contains-an-invalid-ke/32762413#32762413)

## 将打包过程脚本化

工作中，特别是所做项目进入测试阶段，肯定会经常打 Ad-hoc 包给测试人员进行测试，但是我们肯定不想每次进行打包的时候都要进行一些工程的设置修改，以及一系列的 next 按钮点击操作，现在就让这些操作都交给脚本化吧。

1. 脚本化中使用如下的命令打包:

`xcodebuild -project name.xcodeproj -target targetname -configuration Release -sdk iphoneos build CODE_SIGN_IDENTITY="$(CODE_SIGN_IDENTITY)" PROVISIONING_PROFILE="$(PROVISIONING_PROFILE)"`

或者

`xcodebuild -workspace name.xcworkspace -scheme schemename -configuration Release -sdk iphoneos build CODE_SIGN_IDENTITY="$(CODE_SIGN_IDENTITY)" PROVISIONING_PROFILE="$(PROVISIONING_PROFILE)"`

1. 然后使用 xcrun 生成 ipa 文件:

`xcrun -sdk iphoneos -v PackageApplication ./build/Release-iphoneos/$(target|scheme).app"`

1. 清除 build 过程中产生的中间文件
2. 结合蒲公英分发平台，将 ipa 文件上传至蒲公英分发平台，同时在终端会打印上传结果以及上传应用后该应用的 URL。蒲公英分发平台能够方便地将 ipa 文件尽快分发到测试人员，该平台有开放 API，可避免人工上传。

该脚本的使用可使用 `python autobuild.py -h` 查看，与 `xcodebuild` 的使用相似:

> Usage: autobuild.py [options]
>
> Options:
> `-h, --help`: show this help message and exit
> `-w name.xcworkspace, --workspace=name.xcworkspace`: Build the workspace name.xcworkspace.
> `-p name.xcodeproj, --project=name.xcodeproj`: Build the project name.xcodeproj.
> `-s schemename, --scheme=schemename`: Build the scheme specified by schemename. Required if building a workspace.
> `-t targetname, --target=targetname`: Build the target specified by targetname. Required if building a project.
> `-o output_filename, --output=output_filename`: specify output filename

在脚本顶部，有几个全局变量，根据自己的项目情况修改。

```objective-c
CODE_SIGN_IDENTITY = "iPhone Distribution: companyname (9xxxxxxx9A)"
PROVISIONING_PROFILE = "xxxxx-xxxx-xxx-xxxx-xxxxxxxxx"
CONFIGURATION = "Release"
SDK = "iphoneos"

USER_KEY = "15d6xxxxxxxxxxxxxxxxxx"
API_KEY = "efxxxxxxxxxxxxxxxxxxxx"

```

其中，`CODE_SIGN_IDENTITY` 为开发者证书标识，可以在 Keychain Access -> Certificates -> 选中证书右键弹出菜单 -> Get Info -> Common Name 获取，类似 `iPhone Distribution: Company name Co. Ltd (xxxxxxxx9A)`, 包括括号内的内容。

`PROVISIONING_PROFILE`: 这个是 mobileprovision 文件的 identifier，获取方式：

Xcode -> Preferences -> 选中申请开发者证书的 Apple ID -> 选中开发者证书 -> View Details... -> 根据        Provisioning Profiles 的名字选中打包所需的 mobileprovision 文件 -> 右键菜单 -> Show in Finder -> 找到该文件后，除了该文件后缀名的字符串就是 `PROVISIONING_PROFILE` 字段的内容。

当然也可以使用脚本获取, 此处参考 [命令行获取mobileprovision文件的UUID](https://link.jianshu.com?t=http://my.oschina.net/ioslighter/blog/494342):

```objective-c
#!/bin/bash
if [ $# -ne 1 ]
then
  echo "Usage: getmobileuuid the-mobileprovision-file-path"
  exit 1
fi

mobileprovision_uuid=`/usr/libexec/PlistBuddy -c "Print UUID" /dev/stdin <<< $(/usr/bin/security cms -D -i $1)`
echo "UUID is:"
echo ${mobileprovision_uuid}

```

`USER_KEY`, `API_KEY`: 是蒲公英开放 API 的密钥。

先进入*autobuild*目录，使用脚本打包的命令如下:

`python autobuild.py -p ../AOP.xcodeproj -s AOP`

脚本执行完毕，若成功，则会在桌面生成*ipa*文件。

若是打包*xcworkspace*项目，则打包命令格式如下所示:

`python autobuild.py -w ../yourworkspace.xcworkspace -s yourscheme`

常见问题:

1. 找不到*request module*.

```objective-c
import requests
ImportError: No module named requests
```

找不到*request module*， 使用 `$ sudo pip install requests`或者`sudo easy_install -U requests`;
```objective-c
Requests is not a built in module (does not come with the default python installation), so you will have to install it:OSX/LinuxUse `$ sudo pip install requests` if you have `pip` installedAlternatively you can also use `sudo easy_install -U requests` if you have `easy_install`installed.
```


2. 如果使用了上传蒲公英，且安装需要密码，请打开脚本，搜一下脚本里的*password*，将其值设置为空。
3. 使用`xcodebuild -exportArchive`替换*PackageApplication*进行打包.
4. 解析传入参数使用*argparse*替换*OptionParser*.
5. 去掉对*PROVISIONING_PROFILE*和*CODE_SIGN_IDENTITY*的配置，请使用*Xcode8*的自动证书管理。
6. 新增*ExportOptions.plist*文件，用于设置导出*ipa*文件的参数，该文件中的可配置参数可使用`xcodebuild --help`查看。
7. 脚本传入参数去掉`--target`和`--output`，*ipa*文件默认会存放在*Desktop*创建诸如*{scheme}{2016-12-28_08-08-10}*格式的文件夹中。

假如你的项目目录如下所示：

```objective-c
|____AOP
| |____AppDelegate.h
| |____AppDelegate.m
| |____Base.lproj
| | |____LaunchScreen.xib
| | |____Main.storyboard
| |____Images.xcassets
| |____Info.plist
| |____main.m
| |____ViewController.h
| |____ViewController.m
|____AOP.xcodeproj
|____autobuild
| |____autobuild.py
| |____exportOptions.plist

```

*exportOptions.plist*文件中的可选配置参数如下:

```objective-c
compileBitcode : Bool

  For non-App Store exports, should Xcode re-compile the app from bitcode? Defaults to YES.

embedOnDemandResourcesAssetPacksInBundle : Bool

  For non-App Store exports, if the app uses On Demand Resources and this is YES, asset   
  packs are embedded in the app bundle so that the app can be tested without a server to   
  host asset packs. Defaults to YES unless onDemandResourcesAssetPacksBaseURL is specified.

iCloudContainerEnvironment

  For non-App Store exports, if the app is using CloudKit, this configures the   
  "com.apple.developer.icloud-container-environment" entitlement. Available options:   
  Development and Production. Defaults to Development.

manifest : Dictionary

  For non-App Store exports, users can download your app over the web by opening your   
  distribution manifest file in a web browser. To generate a distribution manifest, the   
  value of this key should be a dictionary with three sub-keys: appURL, displayImageURL,   
  fullSizeImageURL. The additional sub-key assetPackManifestURL is required when using on demand resources.

method : String

  Describes how Xcode should export the archive. Available options: app-store, ad-hoc,   
  package, enterprise, development, and developer-id. The list of options varies based on   
  the type of archive. Defaults to development.

onDemandResourcesAssetPacksBaseURL : String

  For non-App Store exports, if the app uses On Demand Resources and   
  embedOnDemandResourcesAssetPacksInBundle isn't YES, this should be a base URL specifying   
  where asset packs are going to be hosted. This configures the app to download asset   
  packs from the specified URL.

teamID : String

  The Developer Portal team to use for this export. Defaults to the team used to build the archive.

thinning : String

  For non-App Store exports, should Xcode thin the package for one or more device   
  variants? Available options: <none> (Xcode produces a non-thinned universal app),   
  <thin-for-all-variants> (Xcode produces a universal app and all available thinned   
  variants), or a model identifier for a specific device (e.g. "iPhone7,1"). Defaults to <none>.

uploadBitcode : Bool

  For App Store exports, should the package include bitcode? Defaults to YES.

uploadSymbols : Bool

  For App Store exports, should the package include symbols? Defaults to YES.
```
8. ExportOptions.plist文件可以通过先使用Xcode 9 手动打包你的项目，然后导出，导出的文件夹里会有这个文件，在导出的文件里面直接拖过来使用，直接复制到你持续集成需要的路径中即可。

9. 
   问题概述自己使用 Jenkins 来做 iOS 项目的持续集成，升级 Xcode 9 之后，编译完成之后打包会一直报如下所示的错误：
   >error: exportArchive: "APPNAME.app" requires a provisioning profile with the Push Notifications feature.
   >Error Domain=IDEProvisioningErrorDomain Code=9 ""APPNAME.app" requires a provisioning profile with the Push Notifications feature." UserInfo={NSLocalizedDescription="APPNAME.app" requires a provisioning profile with the Push Notifications feature., NSLocalizedRecoverySuggestion=Add a profile to the "provisioningProfiles" dictionary in your Export Options property list.}
   >** EXPORT FAILED **
   >Failed to build /Users/Tolecen/.jenkins/workspace/APPNAME/build/APPNAME_release.ipa
   >Build step 'Xcode' marked build as failure
   >Finished: FAILURE

   因为 Xcode 9 默认不允许访问钥匙串的内容，必须要设置 allowProvisioningUpdates 才会允许，但是由于 Xcode integration 插件封闭，并不能对其进行修改加上这个属性，所以决定使用 Shell 脚本代替插件。解决方案将 Jenkins 项目里的 Xcode integration 构建步骤去掉，使用下面所示的命令：

   ```objective-c
   xcodebuild -archivePath "/Users/USERNAME/.jenkins/workspace/APPNAME/build/Debug-iphoneos/APPNAME.xcarchive" -project APPNAME.xcodeproj -sdk iphoneos -scheme "SCHEMENAME" -configuration "Debug" archive

   xcodebuild -exportArchive -archivePath "/Users/USERNAME/.jenkins/workspace/APPNAME/build/Debug-iphoneos/BasketballLeague.xcarchive" -exportPath "/Users/USERNAME/.jenkins/workspace/APPNAME/build/APPNAME_debug" -exportOptionsPlist '/Users/USERNAME/.jenkins/workspace/APPNAME/build/ExportOptions.plist' -allowProvisioningUpdates
   ```

   如果是 workspace 的项目，那就将上面第一段的命令中 -project APPNAME.xcodeproj 修改为 -workspace APPNAME.xcworkspace 即可。

10. Permission denied

   ```objective-c
   原因：当前开发帐号对项目目录没有足够的权限解决： 打开终端，输入命令 sudo chmod -R 777 工作目录 sudo chmod -R 777 /Users/路径

   把py文件的最上面2句话改成
   # !/bin/bash
   #coding:utf-8(解决中文编码问题)
   ```

   ​

