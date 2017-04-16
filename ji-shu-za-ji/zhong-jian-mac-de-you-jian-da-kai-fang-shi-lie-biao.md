
Mac上选中一个文件点击右键会有一个“Open With”菜单，如果你装过多个虚拟机等，这个菜单会出现很多windows系统的程序，从而使这个菜单十分混乱，可以使用以下命令行来清除所有的重复项。
```
/System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/LaunchServices.framework/Versions/A/Support/lsregister -kill -r -domain local -domain user;killall Finder;echo “Open With has been rebuilt, Finder will relaunch“
```

