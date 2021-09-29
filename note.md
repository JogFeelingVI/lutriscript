#### how to delete game list
- path ~/.local/share/lutris
- sqlite3 pga.db
- .tables
- .schema games;
- select * from games;
- delete from games where id=7;
- .quit.

#### Find COMMAND
```
find ~/.local -name 'LD*.desktop' 2>/dev/null -exec rm {} \;
```
> FIND 无权访问,上面命令用于搜索lutris安装过程中,创建的快捷方式
> Tips wine 快捷方式有两种 desktop lnk

#### Install mangoHud and Edit Conf file
> mangohud use steam & lutris
> OpenCL & Vulkan
> gamemoderun MANGOHUD=1 %command%

1. git clone --recurse-submodules https://github.com/flightlessmango/MangoHud.git
2. cd Mangohud
3. poetry init
4. poetry add meson & mako
5. install GOverlay

#### STEAM Not DRM

```
steam_appid.txt Games
这些游戏通常需要运行Steam客户端，但它们可以通过创建一个名为steam_appid.txt的文本文件来实现无DRM，该文件包含了相应游戏的Steam应用ID号，除此之外没有其他内容。steam_appid.txt文件应该和游戏的主程序文件放在同一个文件夹中。
```

#### Tips
```
As I mentioned earlier, that option isn’t actually “switching on discrete GPU”, but “switching from default GPU to secondary”. So in the cases where Intel is the secondary one it’s the one being turned on.
```
