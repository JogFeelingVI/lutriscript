
## Installer documentation

####  Table of contents
```
*  Basics
*  Variable substitution
*  Game configuration directives
*  Runner configuration directives
*  System configuration directives
*  Fetching required files
*  Installer meta data
*  Writing the installation script
*  Example scripts
```

#### Basics

Games in Lutris are written in the YAML format in a declarative way. The same document provides information on how to aquire game files, setup the game and store a base configuration.
> Lutris中的游戏是以YAML格式以声明的方式编写的。同一份文件提供了关于如何获取游戏文件、设置游戏和存储基本配置的信息。

Make sure you have some level of understanding of the YAML format before getting into Lutris scripting. The Ansible documentation provides a short guide on the syntax: https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html

At the very least, a lutris installer should have a game section. If the installer needs to download or ask the user for some files, those can be added in the files section.

Installer instructions are stored in the installer section. This is where the installer files are processed and will results in a runnable game when the installer has finished.

The configuration for a game is constructed from its installer. The files and installer sections are removed from the script, some variables such as $GAMEDIR are substituted and the results is saved in: ~/.config/lutris/games/<game>-<timestamp>.yml.
> 一个游戏的配置是由其安装程序构建的。文件和安装程序部分被从脚本中删除，一些变量如$GAMEDIR被替换，结果被保存在: ~/.config/lutris/games/<game>-<timestamp>.yml。

Published installers can be accessed from a command line by using the lutris: url prefix followed by the installer slug. For example, calling `lutris lutris:quake-darkplaces` will launch the Darkplaces installer for Quake.

Important note: Installer scripts downloaded to the client are embedded in another document. What is editable on the Lutris section is the script section of a bigger document. In addition to the script it self, lutris needs to know the following information:
> 重要提示：下载到客户端的安装程序脚本被嵌入到另一个文件中。在Lutris部分可编辑的是一个更大的文件的脚本部分。除了脚本本身之外，Lutris还需要知道以下信息。

```
name: 游戏的名称，如果包含特殊字符，应该用引号包围。
game_slug: lutris网站上的游戏标识符版本。安装程序的名称
slug: 安装程序的标识符
runner: 用于游戏的运行器。
```
If you intend to write installers locally and not use the website, you should have those keys provided at the root level and everything else indented under a script section. Local installers can be launched from the CLI with lutris -i /path/to/file.yaml.
> 如果你打算在本地编写安装程序，而不使用网站，你应该在根层提供这些键，其他的都缩进到脚本部分。本地安装程序可以在 CLI 中用 `lutris -i /path/to/file.yaml` 启动。
变量替换

#### Variable substitution

You can use variables in your script to customize some aspect of it. Those variables get substituted for their actual value during the install process.

Available variables are:

```
可用的变量有。
$GAMEDIR：安装游戏的绝对路径。
$CACHE: 用于操作游戏文件的临时缓存，在安装结束后删除。
$RESOLUTION: 用户主显示器的全部分辨率（例如：1920x1080）。
$RESOLUTION_WIDTH: 用户主显示器的分辨率宽度（例如：1920）。
$resolution_height: 用户主显示器的分辨率高度(例如：1080)
```

You can also reference files from the files section by their identifier, they will resolve to the absolute path of the downloaded or user provided file. Referencing game files usually doesn't require preceding the variable name with a dollar sign.

#### Installer meta data

> 你也可以通过标识符引用文件部分的文件，它们将解析为下载或用户提供的文件的绝对路径。引用游戏文件通常不需要在变量名前加上美元符号。
安装程序元数据

Installer meta-data is any directive that is at the root level of the installer used for customizing the installer.

Referencing the main file

Referencing the main file of a game is possible to do at the root level of the installer but this information is later merged in the game section. It is recommended to put this information directly in the game section. If you see an existing installer with keys like exe or main_file sitting at the root level, please move them to the game section.
> 引用游戏的主文件可以在安装程序的根层进行，但这一信息后来被合并到了游戏部分。我们建议直接把这些信息放在游戏部分。如果你看到现有的安装程序在根层有exe或main_file等键，请将它们移到游戏部分。

Customizing the game's name

Use the custom-name directive to override the name of the game. Use this only if the installer provides a significantly different game from the base one.

Example: custom-name: Quake Champions: Doom Edition

Requiring additional binaries

If the game or the installer needs some system binaries to run, you can specify them in the require-binaries directive. The value is a comma-separated list of required binaries (acting as AND), if one of several binaries are able to run the program, you can add them as a | separated list (acting as OR).
>如果游戏或安装程序需要一些系统二进制文件来运行，你可以在require-binaries指令中指定它们。该值是一个用逗号分隔的所需二进制文件的列表（充当AND），如果几个二进制文件中的一个能够运行程序，你可以把它们作为一个|分隔的列表加入（充当OR）。
Example:
```
# This requires cmake to be installed and either ggc or clang
require-binaries: cmake, gcc | clang
```
Mods and add-ons

Mods and add-ons require that a base game is already installed on the system. You can let the installer know that you want to install an add-on by specifying the requires directive. The value of requires must be the canonical slug name of the base game, not one of its aliases. For example, to install the add-on "The reckoning" for Quake 2, you should add: requires: quake-2
> Mods和add-ons要求系统上已经安装了基本游戏。你可以通过指定 requires 指令让安装程序知道你想安装一个附加组件。requires的值必须是基本游戏的标准名称，而不是它的一个别名。例如，要安装Quake 2的附加组件 "The reckoning"，你应该添加： requires: quake-2

You can also add complex requirements following the same syntax as the require-binaries directive described above.

Customizing the end of install text

You can display a custom message when the installation is completed. To do so, use the install_complete_text key.
> 你可以在安装完成后显示一个自定义的信息。要做到这一点，请使用 install_complete_text 键。

#### Game configuration directives

A game configuration file can contain up to 3 sections: game, system and a section named after the runner used for the game.

The game section can also contain references to other stores such as Steam or GOG. Some IDs are used to launch the game (Steam, ScummVM) while in other cases, the ID is only used to find games files on a 3rd party platform and download the installer (Humble Bundle, GOG).
> 游戏部分也可以包含对其他商店的引用，如Steam或GOG。有些ID用于启动游戏（Steam、ScummVM），而在其他情况下，ID只用于在第三方平台上寻找游戏文件和下载安装程序（Humble Bundle、GOG）。

Lutris supports the following game identifiers:

appid: For Steam games. Numerical ID found in the URL of the store page. Example: The appid for https://store.steampowered.com/app/238960/Path_of_Exile/ is 238960. This ID is used for installing and running the game.

game_id: Identifier used for ScummVM / ResidualVM games. Can be looked up on the game compatibility list: https://www.scummvm.org/compatibility/ and https://www.residualvm.org/compatibility/

gogid: GOG identifier. Can be looked up on https://www.gogdb.org/products Be sure to reference the base game and not one of its package or DLC. Example: The gogid for Darksiders III is 1246703238

humbleid: Humble Bundle ID. There currently isn't a way to lookup game IDs other than using the order details from the HB API. Lutris will soon provide easier ways to find this ID.

main_file: For MAME games, the main_file can refer to a MAME ID instead of a file path.
Common game section entries
> main_file。对于MAME游戏，main_file可以指一个MAME ID而不是文件路径。
常见的游戏部分条目

exe: Main game executable. Used for Linux and Wine games. Example: exe: exult
> exe。主要的游戏可执行文件。用于Linux和Wine游戏。例如：Exe: exult

main_file: Used in most emulator runners to reference the ROM or disk file. Example: main_file: game.rom. Can also be used to pass the URL for web based games: main_file: http://www...
> main_file。在大多数仿真器运行程序中用来引用ROM或磁盘文件。例如：main_file: game.rom。也可以用来传递基于网络的游戏的URL：main_file: http://www...

args: Pass additional arguments to the command. Can be used with linux, wine, winesteam, dosbox, scummvm, pico8 and zdoom runners. Example: args: -c $GAMEDIR/exult.cfg
> args。向命令传递额外的参数。可用于linux、wine、winesteam、dosbox、scummvm、pico8和zdoom运行器。例如：args: -c $GAMEDIR/exult.cfg

working_dir: Set the working directory for the game executable. This is useful if the game needs to run from a different directory than the one the executable resides in. This directive can be used for Linux, Wine and Dosbox installers. Example: $GAMEDIR/path/to/game
Wine and other wine based runners like winesteam
> working_dir: 设置游戏可执行文件的工作目录。如果游戏需要从与可执行文件所在的目录不同的目录中运行，这一点很有用。这个指令可以用于Linux、Wine和Dosbox安装程序。例如：$GAMEDIR/path/to/game
wine和其他基于wine的运行器，如winesteam

arch: Sets the architecture of a Wine prefix. By default it is set to win64, the value can be set to win32 to setup the game in a 32-bit prefix.
> arch: 设置Wine前缀的架构。默认情况下，它被设置为win64，该值可以被设置为win32以在32位前缀中设置游戏。

prefix: Path to the Wine prefix. For Wine games, it should be set to $GAMEDIR. For WineSteam games, set it to $GAMEDIR/prefix to isolate the prefix files from the the game files. This is only needed if the Steam game needs customization. If not provided, Lutris will use Winesteam's default prefix where Steam for Windows is installed.
> prefix。到Wine前缀的路径。对于Wine游戏，它应该被设置为$GAMEDIR。对于WineSteam游戏，将其设置为$GAMEDIR/prefix，以便将前缀文件与游戏文件隔离。只有在Steam游戏需要定制的时候才需要这样做。如果不提供，Lutris将使用Winesteam的默认前缀，其中Steam for Windows已经安装。

无DRM的Steam和WineSteam

Lutris has the ability to run Steam games without launching the Steam client. This is only possible with certain games lacking the Steam DRM.

run_without_steam: Activate the DRM free mode and no not launch Steam when the game runs.

steamless_binary: Used in conjonction with run_without_steam. This allows to provide the path of the game executable if it's able to run without the Steam client. The game must not have the Steam DRM to use this feature.
> Lutris有能力在不启动Steam客户端的情况下运行Steam游戏。这只适用于某些缺乏Steam DRM的游戏。
run_without_steam: 激活无DRM模式，在游戏运行时不启动Steam。
无steam_binary。与run_without_steam结合使用。这允许提供游戏可执行文件的路径，如果它能够在没有Steam客户端的情况下运行。游戏必须没有Steam的DRM才能使用这个功能。
Example: steamless_binary: $GAMEDIR/System/GMDX.exe

ScummVM

path: Location of the game files. This should be set to $GAMEDIR in installer scripts.
> 游戏文件的位置。在安装脚本中，这应该被设置为$GAMEDIR。
#### Runner configuration directives

Runners can be customized in a section named after the runner identifier (slug field in the API). A complete list of all runners is available at https://lutris.net/api/runners. Use the runner's slug as the runner identifier. Please keep the amount of runner customization to a minimum, only adding what is needed to make the game run correctly. A lot of runner options do not have their place in Lutris installers and are reserved for the user's preferences.

The following sections will describe runner directives commonly used in installers.

`wine`

version: Set the Wine version to a specific build. Only set this if the game has known regressions with the current default build. Abusing this feature slows down the development of the Wine project. Example: version: staging-2.21-x86_64

`Desktop`: Run the game in a Wine virtual desktop. This should be used if the game has issues with Linux window managers such as crashes on Alt-Tab. Example: Desktop: true
> 在Wine虚拟桌面中运行游戏。如果游戏在Linux窗口管理器中出现问题，例如在Alt-Tab下崩溃，就应该使用这个方法。

WineDesktop: Set the resolution of the Wine virtual desktop. If not provided, the virtual desktop will take up the whole screen, which is likely the desired behavior. It is unlikely that you would add this directive in an installer but can be useful is a game is picky about the resolution it's running in. Example: WineDesktop: 1024x768
> WineDesktop。设置Wine虚拟桌面的分辨率。如果不提供，虚拟桌面将占满整个屏幕，这可能是人们所期望的行为。你不太可能在安装程序中添加这个指令，但如果游戏对它运行的分辨率很挑剔的话，这个指令就很有用。例子。WineDesktop: 1024x768

dxvk: Use this to disable DXVK if needed. (dxvk: false)

esync: Use this to enable esync. (esync: true)

overrides: Overrides for Wine DLLs. List your DLL overrides in a mapping with the following values:
> wine DLL的重写。在映射中列出你的DLL重写，其值如下。

n,b = Try native and fallback to builtin if native doesn't work

b,n = Try builtin and fallback to native if builtin doesn't work

b = Use builtin 使用内建的

n = Use native 使用本地的

disabled = Disable library
```
  Example:
  overrides:
  ddraw.dll: n
  d3d9: disabled
  winegstreamer: builtin
```  

#### System configuration directives

Those directives are stored in the system section and allow for customization of system features. As with runner configuration options, system directives should be used carefully, only adding them when absolutely necessary to run a game.

restore_gamma: If the game doesn't restore the correct gamma on exit, you can use this option to call xgamma and reset the default values. This option won't work on Wayland. Example: restore_gamma: true
> restore_gamma: 如果游戏在退出时没有恢复正确的伽马值，你可以使用这个选项调用xgamma并重置默认值。这个选项在Wayland上不起作用。Example: restore_gamma: true

terminal: Run the game in a terminal if the game is a text based one. Do not use this option to get the console output of the game, this will result in a broken installer. Only use this option for text based games.
> 如果游戏是基于文本的，在终端中运行游戏。不要使用这个选项来获取游戏的控制台输出，这将导致安装程序损坏。只对基于文本的游戏使用该选项。


env: Sets environment variables before launching a game and during install. Do not ever use this directive to enable a framerate counter. Do not use this directive to override Wine DLLS. Variable substitution is available in values. 
> 在启动游戏前和安装过程中设置环境变量。千万不要使用这个指令来启用帧速率计数器。不要使用此指令来覆盖Wine DLLS。变量替换在数值中是可用的。
```
Example:
  env:
    __GL_SHADER_DISK_CACHE: 1
    __GL_THREADED_OPTIMIZATIONS: '1'
    __GL_SHADER_DISK_CACHE_PATH: $GAMEDIR
    mesa_glthread: 'true'
```

single_cpu: Run the game on a single CPU core. Useful for some old games that handle multicore CPUs poorly. (single_cpu: true)
> 在单个CPU核心上运行游戏。对一些处理多核CPU较差的老游戏很有用。(single_cpu: true)

disable_runtime: DO NOT DISABLE THE LUTRIS RUNTIME IN LUTRIS INSTALLERS
> 不要在 lutris 安装程序中禁用 lutris 运行时间。

pulse_latency: Set PulseAudio latency to 60 msecs. Can reduce audio stuttering. (pulse_latency: true)
> 将PulseAudio的延迟设置为60毫秒。可以减少音频的停顿。(pulse_latency: true)

use_us_layout: Change the keyboard layout to a standard US one while the game is running. Useful for games that handle other layouts poorly and don't have key remapping options. (use_us_layou: true)
> 在游戏运行时将键盘布局改为标准的美国键盘布局。对于那些对其他布局处理不力、没有按键重映射选项的游戏很有用。(use_us_layou: true)

xephyr: Run the game in Xephyr. This is useful for games only handling 256 color modes. To enable Xephyr, pass the desired bit per plane value. (xephyr: 8bpp)
> 在Xephyr中运行游戏。这对只处理256色模式的游戏很有用。要启用Xephyr，需要传递每个平面所需的比特值。(xephyr: 8bpp)

xephyr_resolution: Used with the xephyr option, this sets the size of the Xephyr window. (xephyr_resolution: 1024x768)
> 与xephyr选项一起使用，它设置Xephyr窗口的大小。(xephyr_resolution: 1024x768)

#### Fetching required files

The files section of the installer references every file needed for installing the game. This section's keys are unique identifier used later in the installer section. The value can either be a string containing a URI pointing at the required file or a dictionary containing the filename and url keys. The url key is equivalent to passing only a string to the installer and the filename key will be used to give the local copy another name. If you need to set referer use referer key.

If the game contains copyrighted files that cannot be redistributed, the value should begin with N/A. When the installer encounter this value, it will prompt the user for the location of the file. To indicate to the user what file to select, append a message to N/A like this: N/A:Please select the installer for this game

> 安装程序的文件部分引用了安装游戏所需的每个文件。这一部分的键是安装程序部分后面使用的唯一标识符。该值可以是一个包含指向所需文件的URI的字符串，也可以是一个包含文件名和url键的字典。url键相当于只向安装程序传递一个字符串，而文件名键将被用来给本地拷贝提供另一个名字。如果你需要设置referer，请使用referer键。
> 如果游戏中包含有版权的文件，不能重新分发，那么这个值应该以N/A开头。当安装程序遇到这个值时，它将提示用户文件的位置。要确定
```
Examples:
files:
- file1: https://example.com/gamesetup.exe
- file2: "N/A:Select the game's setup file"
- file3:
    url: https://example.com/url-that-doesnt-resolve-to-a-proper-filename
    filename: actual_local_filename.zip
    referer: www.mywebsite.com
```

If the game makes use of (Windows) Steam data, the value should be $WINESTEAM:appid:path/to/data. This will check that the data is available or install it otherwise.
> 如果游戏使用了（Windows）Steam数据，其值应该是$WINESTEAM:appid:path/to/data。这将检查该数据是否可用，否则就安装它。

#### Writing the installation script

After every file needed by the game has been aquired, the actual installation can take place. A series of directives will tell the installer how to set up the game correctly. Start the installer section with installer: then stack the directives by order of execution (top to bottom).
Displaying an 'Insert disc' dialog
> 在获得游戏所需的每个文件后，就可以进行实际的安装了。一系列的指令将告诉安装程序如何正确设置游戏。用installer: 开始安装程序部分，然后按执行顺序（从上到下）堆叠指令。 显示 "插入光盘 "对话框

The insert-disc command will display a message box to the user requesting him to insert the game's disc into the optical drive.
> insert-disc命令将向用户显示一个消息框，要求他将游戏的光盘插入光驱。

Ensure a correct disc detection by specifying a file or folder present on the disc with the requires parameter.
> 通过在requirements参数中指定光盘上的文件或文件夹来确保正确的光盘检测。

The $DISC variable will contain the drive's path for use in subsequent installer tasks.
> $DISC变量将包含驱动器的路径，以便在随后的安装任务中使用。

A link to CDEmu's homepage and PPA will also be displayed if the program isn't detected on the machine, otherwise it will be replaced with a button to open gCDEmu. You can override this default text with the message parameter.
> 如果机器上没有检测到该程序，也会显示一个指向CDEmu主页和PPA的链接，否则会被一个打开gCDEmu的按钮取代。你可以用信息参数覆盖这个默认文本。

```
Example:
- insert-disc:
    requires: diablosetup.exe
```

Moving files and directories

Move files or directories by using the move command. move requires two parameters: src (the source file or folder) and dst (the destination folder).
> 使用move命令来移动文件或目录。move需要两个参数：src（源文件或文件夹）和dst（目标文件夹）。

The src parameter can either be a file ID or a path relative to game dir. If the parameter value is not found in the list of file ids, then it must be prefixed by either $CACHE or $GAMEDIR to move a file or directory from the download cache or the game's install dir, respectively.
> src参数可以是一个文件ID，也可以是相对于游戏dir的一个路径。如果在文件ID列表中找不到参数值，那么必须在参数前加上$CACHE或$GAMEDIR，以便分别从下载缓存或游戏安装目录中移动文件或目录。


The dst parameter should be prefixed by either $GAMEDIR or $HOME to move files to path relative to the game dir or the current user's home
> dst参数应该以$GAMEDIR或$HOME为前缀，以便将文件移动到与游戏目录或当前用户的主页相对的路径上。

If the source is a file ID, it will be updated with the new destination path. It can then be used in following commands to access the moved file.
> 如果源是一个文件ID，它将被更新为新的目标路径。然后可以在下面的命令中使用它来访问被移动的文件。

The move command cannot overwrite files.

```
Example:
- move:
    src: game_file_id
    dst: $GAMEDIR/location
```

Copying and merging directories

Both merging and copying actions are done with the merge or the copy directive. It is not important which of these directives is used because copy is just an alias for merge. Whether the action does a merge or copy depends on the existence of the destination directory. When merging into an existing directory, original files with the same name as the ones present in the merged directory will be overwritten. Take this into account when writing your script and order your actions accordingly.
> 合并和复制的动作都是通过merge或copy指令完成的。使用哪一个指令并不重要，因为copy只是merge的一个别名。这个动作是做合并还是复制，取决于目标目录的存在。当合并到一个现有的目录中时，与被合并目录中的文件名称相同的原始文件将被覆盖。在编写你的脚本时要考虑到这一点，并相应地安排你的动作。

If the source is a file ID, it will be updated with the new destination path. It can then be used in following commands to access the copied file.
> 如果源是一个文件ID，它将被更新为新的目标路径。然后可以在下面的命令中使用它来访问复制的文件。

```
Example:
- merge:
    src: game_file_id
    dst: $GAMEDIR/location
```

Extracting archives

Extracting archives is done with the extract directive, the file argument is a file id or a file path with optional wildcards. If the archive(s) should be extracted in some other location than the $GAMEDIR, you can specify a dst argument.
> 提取归档文件是通过提取指令完成的，文件参数是一个文件ID或一个带有可选通配符的文件路径。如果归档文件应该在$GAMEDIR以外的其他位置被提取，可以指定一个dst参数。

You can optionally specify the archive's type with the format option. This is useful if the archive's file extension does not match what it should be. Accepted values for format are: zip, tgz, gzip, bz2, and gog (innoextract).
> 你可以选择用format选项来指定存档的类型。如果归档文件的扩展名与它应该是的类型不一致，这就很有用。接受的格式值有：zip、tgz、gzip、bz2和gog（innoextract）。
```
Example:
- extract:
    file: game_archive
    dst: $GAMEDIR/datadir/
```

Making a file executable

Marking the file as executable is done with the chmodx directive. It is often needed for games that ship in a zip file, which does not retain file permissions.
> 将文件标记为可执行文件是通过chmodx指令完成的。对于以压缩文件形式运送的游戏来说，这通常是需要的，因为压缩文件不保留文件权限。


Example: - chmodx: $GAMEDIR/game_binary
Executing a file

Execute files with the execute directive. Use the file parameter to reference a file id or a path, args to add command arguments, terminal (set to "true") to execute in a new terminal window, working_dir to set the directory to execute the command in (defaults to the install path). The command is executed within the Lutris Runtime (resolving most shared library dependencies). The file is made executable if necessary, no need to run chmodx before. You can also use env (environment variables), exclude_processes (space-separated list of processes to exclude from being watched), include_processes (the opposite of exclude_processes, is used to override Lutris' built-in exclude list) and disable_runtime (run a process without the Lutris Runtime, useful for running system binaries).
> 用execute指令执行文件。使用文件参数来引用一个文件ID或路径，使用args来添加命令参数，使用terminal（设置为 "true"）来在新的终端窗口中执行，使用working_dir来设置执行命令的目录（默认为安装路径）。该命令在Lutris Runtime中执行（解决大多数共享库的依赖性）。如果有必要，该文件将被变成可执行文件，不需要在之前运行chmodx。你也可以使用 env（环境变量）、exclude_processes（用空格分隔的进程列表，以排除被监视的进程）、include_processes（与exclude_processes相反，用于覆盖 Lutris 的内置排除列表）和 disable_runtime（运行一个没有 Lutris Runtime 的进程，对于运行系统二进制文件很有用）。

```
Example:
- execute:
    args: --argh
    file: great_id
    terminal: true
    env:
    *   key: value
```

You can use the command parameter instead of file and args. This lets you run bash/shell commands easier. bash is used and is added to include_processes internally.
> 你可以使用命令参数而不是文件和args。这可以让你更容易地运行bash/shell命令。bash被使用，并在内部被添加到include_processes。

```
Example:
- execute:
    command: 'echo Hello World! | cat'
```

Writing files
Writing text files

Create or overwrite a file with the write_file directive. Use the file (an absolute path or a file id) and content parameters.
> 用write_file指令创建或覆盖一个文件。使用文件（绝对路径或文件ID）和内容参数。

You can also use the optional parameter mode to specify a file write mode. Valid values for mode include w (the default, to write to a new file) or a to append data to an existing file.
> 你也可以使用可选的参数mode来指定一个文件的写入模式。模式的有效值包括w（默认值，写入一个新文件）或a，将数据追加到一个现有文件。

Refer to the YAML documentation for reference on how to including multiline documents and quotes.

Example:

- write_file:
    file: $GAMEDIR/myfile.txt
    content: 'This is the contents of the file.'

Writing into an INI type config file

Modify or create a config file with the write_config directive. A config file is a text file composed of key=value (or key: value) lines grouped under [sections]. Use the file (an absolute path or a file id), section, key and value parameters or the data parameter. Set merge: false to first truncate the file. Note that the file is entirely rewritten and comments are left out; Make sure to compare the initial and resulting file to spot any potential parsing issues.
> 用write_config指令修改或创建一个配置文件。配置文件是一个由键=值（或键：值）行组成的文本文件，被分组在[节]下。使用文件（绝对路径或文件ID）、部分、键和值参数或数据参数。设置merge: false来首先截断文件。请注意，文件被完全重写，注释被遗漏；请确保比较初始文件和结果文件，以发现任何潜在的解析问题。

```
Example:
- write_config:
    file: $GAMEDIR/myfile.ini
    section: Engine
    key: Renderer
    value: OpenGL

- write_config:
    file: $GAMEDIR/myfile.ini
    data:
    *   General:
    *     iNumHWThreads: 2
    *     bUseThreadedAI: 1
```
Writing into a JSON type file

Modify or create a JSON file with the write_json directive. Use the file (an absolute path or a file id) and data parameters. Note that the file is entirely rewritten; Make sure to compare the initial and resulting file to spot any potential parsing issues. You can set the optional parameter merge to false if you want to overwrite the JSON file instead of updating it.
> 用write_json指令修改或创建一个JSON文件。使用文件（绝对路径或文件ID）和数据参数。注意，该文件是完全重写的；确保比较初始文件和结果文件，以发现任何潜在的解析问题。如果你想覆盖JSON文件而不是更新它，你可以把可选参数merge设置为false。

```
Example:
- write_json:
    file: $GAMEDIR/myfile.json
    data:
    *   Sound:
    *     Enabled: 'false'

This writes (or updates) a file with the following content:

{
  "Sound": {
    "Enabled": "false"
  }
}
```

Running a task provided by a runner

Some actions are specific to some runners, you can call them with the task command. You must at least provide the name parameter which is the function that will be called. Other parameters depend on the task being called. It is possible to call functions from other runners by prefixing the task name with the runner's name (e.g., from a dosbox installer you can use the wineexec task with wine.wineexec as the task's name)
> 有些动作是特定于某些运行器的，你可以用任务命令调用它们。你至少要提供名称参数，这是将要被调用的函数。其他参数取决于被调用的任务。通过在任务名称前加上运行程序的名称，可以从其他运行程序中调用函数（例如，从dosbox安装程序中，你可以使用wineexec任务，任务名称为wine.wineexec）。

Currently, the following tasks are implemented:

    wine / winesteam: create_prefix Creates an empty Wine prefix at the specified path. The other wine/winesteam directives below include the creation of the prefix, so in most cases you won't need to use the create_prefix command. Parameters are:
*     prefix: the path
*     arch: optional architecture of the prefix, default: win64 unless a 32bit build is specified in the runner options.
*     overrides: optional dll overrides, format described later
*     install_gecko: optional variable to stop installing gecko
*     install_mono: optional variable to stop installing mono

> 目前，已经实现了以下任务。

    wine / winesteam: create_prefix 在指定的路径创建一个空的 Wine 前缀。下面的其他 wine/winesteam 指令包括创建前缀，所以在大多数情况下，你不需要使用 create_prefix 命令。参数是。
    are:
*    prefix: 路径
*    arch：前缀的可选架构，默认为win64，除非在runner选项中指定了32位的构建。
*    overrides: 可选的dll覆盖物，格式稍后描述
*    install_gecko: 可选变量，用于停止安装gecko
*    install_mono: 可选变量，用于停止安装mono。
```      
    Example:
    - task:
*     name: create_prefix
*     arch: win64
```
      
    wine / winesteam: wineexec Runs a windows executable. Parameters are executable (file ID or path), args (optional arguments passed to the executable), prefix (optional WINEPREFIX), arch (optional WINEARCH, required when you created win64 prefix), blocking (if true, do not run the process in a thread), working_dir (optional working directory), include_processes (optional space-separated list of processes to include to being watched) exclude_processes (optional space-separated list of processes to exclude from being watched), env (optional environment variables), overrides (optional dll overrides).

    > wine / winesteam: wineexec 运行一个windows可执行文件。参数是可执行文件（文件ID或路径），args（传递给可执行文件的可选参数），prefix（可选的WINEPREFIX），arch（可选的WINEARCH，当你创建win64前缀时需要），blocking（如果为真，不在一个线程中运行进程），working_dir（可选的工作目录）。include_processes (可选的以空格分隔的进程列表，以包括被监视的进程) exclude_processes (可选的以空格分隔的进程列表，以排除被监视的进程), env (可选的环境变量), overrides (可选的dll覆盖).
    
    Example:
    
    - task:
*     name: wineexec
*     executable: drive_c/Program Files/Game/Game.exe
*     args: --windowed
      
    wine / winesteam: winetricks Runs winetricks with the app argument. prefix is an optional WINEPREFIX path. You can run many tricks at once by adding more to the app parameter (space-separated).
    
    By default Winetricks will run in silent mode but that can cause issues with some components such as XNA. In such cases, you can provide the option silent: false
    
    Example:
    
    - task:
*     name: winetricks
*     app: nt40
      
    For a full list of available winetricks see here: https://github.com/Winetricks/winetricks/tree/master/files/verbs
    
    wine / winesteam: eject_disk runs eject_disk in your prefix argument. Parameters are prefix (optional wineprefix path).
    > wine / winesteam: eject_disk 在你的前缀参数中运行eject_disk。参数是prefix（可选的wineprefix路径）。
    Example:
    
    - task:
*     name: eject_disc
      
    wine / winesteam: set_regedit Modifies the Windows registry. Parameters are path (the registry path, use backslashes), key, value, type (optional value type, default is REG_SZ (string)), prefix (optional WINEPREFIX), arch (optional architecture of the prefix, required when you created win64 prefix).
    > wine / winesteam: set_regedit 修改Windows注册表。参数是path（注册表路径，使用反斜线）、key、value、type（可选的值类型，默认是REG_SZ（字符串））、prefix（可选的WINEPREFIX）、arch（可选的架构前缀，当你创建win64前缀时需要）。
    
    Example:
    
    - task:
*     name: set_regedit
*     path: HKEY_CURRENT_USER\Software\Valve\Steam
*     key: SuppressAutoRun
*     value: '00000000'
*     type: REG_DWORD
      
    wine / winesteam: delete_registry_key Deletes registry key in the Windows registry. Parameters are key, prefix (optional WINEPREFIX), arch (optional architecture of the prefix, required when you created win64 prefix).
    >  wine / winesteam: delete_registry_key 删除Windows注册表中的注册表键。参数是键，前缀（可选WINEPREFIX），arch（可选架构的前缀，当你创建win64前缀时需要）。

    Example:
    
    - task:
*     name: set_regedit
*     path: HKEY_CURRENT_USER\Software\Valve\Steam
*     key: SuppressAutoRun
*     value: '00000000'
*     type: REG_DWORD
      
    wine / winesteam: set_regedit_file Apply a regedit file to the registry, Parameters are filename (regfile name), arch (optional architecture of the prefix, required when you created win64 prefix).
    >  wine / winesteam: set_regedit_file 在注册表中应用一个regedit文件，参数是filename（regfile名称），arch（可选的架构前缀，当你创建win64前缀时需要）。
    Example:
    
    - task:
*     name: set_regedit_file
*     filename: myregfile
      
    wine / winesteam: winekill Stops processes running in Wine prefix. Parameters are prefix (optional WINEPREFIX), arch (optional architecture of the prefix, required when you created win64 prefix).
    > wine / winesteam: winekill 停止运行在Wine前缀的进程。参数是prefix（可选的WINEPREFIX），arch（可选的前缀架构，当你创建win64前缀时需要）。
    Example
    
    - task:
*     name: winekill
      
    dosbox: dosexec Runs dosbox. Parameters are executable (optional file ID or path to executable), config_file (optional file ID or path to .conf file), args (optional command arguments), working_dir (optional working directory, defaults to the executable's dir or the config_file's dir), exit (set to false to prevent DOSBox to exit when the executable is terminated).
    > dosbox: dosexec 运行dosbox。参数是executable（可执行文件ID或可执行文件的路径），config_file（可选的文件ID或.conf文件的路径），args（可选的命令参数），working_dir（可选的工作目录，默认为可执行文件的目录或config_file的目录），exit（设置为false，防止DOSBox在执行程序终止时退出）。
    
    Example:
    
    - task:
*     name: dosexec
*     executable: file_id
*     config: $GAMEDIR/game_install.conf
*     args: -scaler normal3x -conf more_conf.conf

Displaying a drop-down menu with options

Request input from the user by displaying a menu filled with options to choose from with the input_menu directive. The description parameter holds the message to the user, options is an indented list of value: label lines where "value" is the text that will be stored and "label" is the text displayed, and the optional preselect parameter is the value to preselect for the user.
> Request input from the user by displaying a menu filled with options to choose from with the input_menu directive. The description parameter holds the message to the user, options is an indented list of value: label lines where "value" is the text that will be stored and "label" is the text displayed, and the optional preselect parameter is the value to preselect for the user.

The result of the last input directive is available with the $INPUT alias. If need be, you can add an id parameter to the directive which will make the selected value available with $INPUT_<id> with "<id>" obviously being the id you specified. The id must contain only numbers, letters and underscores.
> 最后一条输入指令的结果可以用$INPUT的别名来表示。如果需要，你可以在指令中添加一个id参数，这样就可以用$INPUT_<id>来获得所选的值，"<id>"显然就是你指定的id。该id必须只包含数字、字母和下划线。

Example:

- input_menu:
    description: "Choose the game's language:"
    id: LANG
    options:
    - en: English
    - fr: French
    - "value and": "label can be anything, surround them with quotes to avoid issues"
    preselect: en

In this example, English would be preselected. If the option eventually selected is French, the "$INPUT_LANG" alias would be available in following directives and would correspond to "fr". "$INPUT" would work as well, up until the next input directive.
> 在这个例子中，英语将被预选。如果最终选择的是法语，"$INPUT_LANG "这个别名将在接下来的指令中可用，并对应于 "fr"。"$INPUT "也可以工作，直到下一个输入指令。

#### Example scripts

Those example scripts are intended to be used as standalone files. Only the script section should be added to the script submission form.

Example Linux game:

name: My Game
game_slug: my-game
version: Installer
slug: my-game-installer
runner: linux

script:
  game:
    exe: $GAMEDIR/mygame
    args: --some-arg
    working_dir: $GAMEDIR

  files:
  - myfile: https://example.com/mygame.zip

  installer:
  - chmodx: $GAMEDIR/mygame
    system:
    terminal: true
    env:
    *   SOMEENV: true

Example wine game:

name: My Game
game_slug: my-game
version: Installer
slug: my-game-installer
runner: wine

script:
  game:
    exe: $GAMEDIR/mygame
    args: --some-args
    prefix: $GAMEDIR/prefix
    arch: win32
    working_dir: $GAMEDIR/prefix
  files:
  - installer: "N/A:Select the game's setup file"
    installer:
  - task:
*   executable: installer
*   name: wineexec
*   prefix: $GAMEDIR/prefix
    wine:
    Desktop: true
    overrides:
*   ddraw.dll: n
    system:
    terminal: true
    env:
*   WINEDLLOVERRIDES: d3d11=
*   SOMEENV: true

Example gog wine game, some installer crash with with /SILENT or /VERYSILENT option (Cuphead and Star Wars: Battlefront II for example), (most options can be found here http://www.jrsoftware.org/ishelp/index.php?topic=setupcmdline, there is undocumented gog option /NOGUI, you need to use it when you use /SILENT and /SUPPRESSMSGBOXES parameters):

name: My Game
game_slug: my-game
version: Installer
slug: my-game-installer
runner: wine

script:
  game:
    exe: $GAMEDIR/drive_c/game/bin/Game.exe
    args: --some-arg
    prefix: $GAMEDIR
    working_dir: $GAMEDIR/drive_c/game
  files:
  - installer: "N/A:Select the game's setup file"
    installer:
  - task:
*   args: /SILENT /LANG=en /SP- /NOCANCEL /SUPPRESSMSGBOXES /NOGUI /DIR="C:/game"
*   executable: installer
*   name: wineexec
    wine:
    Desktop: true
    overrides:
*   ddraw.dll: n
    system:
    terminal: true
    env:
*   SOMEENV: true

Example gog wine game, alternative (requires innoextract):

name: My Game
game_slug: my-game
version: Installer
slug: my-game-installer
runner: wine

script:
  game:
    exe: $GAMEDIR/drive_c/Games/YourGame/game.exe
    args: --some-arg
    prefix: $GAMEDIR/prefix
  files:
  - installer: "N/A:Select the game's setup file"
    installer:
  - execute:
*   args: --gog -d "$CACHE" setup
*   description: Extracting game data
*   file: innoextract
  - move:
*   description: Extracting game data
*   dst: $GAMEDIR/drive_c/Games/YourGame
*   src: $CACHE/app
    wine:
    Desktop: true
    overrides:
*   ddraw.dll: n
    system:
    env:
*   SOMEENV: true

Example gog linux game (mojosetup options found here https://www.reddit.com/r/linux_gaming/comments/42l258/fully_automated_gog_games_install_howto/):

name: My Game
game_slug: my-game
version: Installer
slug: my-game-installer
runner: linux

script:
  game:
    exe: $GAMEDIR/game.sh
    args: --some-arg
    working_dir: $GAMEDIR
  files:
  - installer: "N/A:Select the game's setup file"
    installer:
  - chmodx: installer
  - execute:
    *   file: installer
    *   description: Installing game, it will take a while...
    *   args: -- --i-agree-to-all-licenses --noreadme --nooptions --noprompt --destination=$GAMEDIR
    system:
    terminal: true

Example gog linux game, alternative:

name: My Game
game_slug: my-game
version: Installer
slug: my-game-installer
runner: linux

script:
  files:
  - goginstaller: N/A:Please select the GOG.com Linux installer
    game:
    args: --some-arg
    exe: start.sh
    installer:
  - extract:
*   dst: $CACHE/GOG
*   file: goginstaller
*   format: zip
  - merge:
    *   dst: $GAMEDIR
    *   src: $CACHE/GOG/data/noarch/
    system:
    terminal: true

Example winesteam game:

name: My Game
game_slug: my-game
version: Installer
slug: my-game-installer
runner: winesteam

script:
  game:
    appid: 227300
    args: --some-args
    prefix: $GAMEDIR/prefix
    arch: win64
  installer:
  - task:
*   description: Setting up wine prefix
*   name: create_prefix
*   prefix: $GAMEDIR/prefix
*   arch: win64
    winesteam:
    Desktop: true
    WineDesktop: 1024x768
    overrides:
*   ddraw.dll: n

Example steam Linux game:

name: My Game
game_slug: my-game
version: Installer
slug: my-game-installer
runner: steam

script:
  game:
    appid: 227300
    args: --some-args

