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

#### Tips
```
As I mentioned earlier, that option isn’t actually “switching on discrete GPU”, but “switching from default GPU to secondary”. So in the cases where Intel is the secondary one it’s the one being turned on.
```
