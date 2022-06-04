# Anaconda: 中文路径

有时我们的 Windows 系统电脑的用户名设置成了中文 (比如我), 这样我们的 Appdata 保存的 `user` 路径就会有中文. 在安装包时可能会出现如下报错:

```
InvalidArchiveError("Error with archive C:\\Users\\一些中文\\.conda\\pkgs\\matplotlib-3.5.1-py39haa95532_1iizru2mc\\info-matplotlib-3.5.1-py39haa95532_1.tar.zst.  You probably need to delete and re-download or re-create this file.  Message from libarchive was:\n\nFailed to open 'C:\\Users\\一些中文\\.conda\\pkgs\\matplotlib-3.5.1-py39haa95532_1iizru2mc\\info-matplotlib-3.5.1-py39haa95532_1.tar.zst'")
```

此时你可能会非常痛苦, 因为这个用户名是不能修改的, 即使你想修改，就会发现这个文件夹的右键菜单根本就没有 "重命名"!

解决方法很简单. 就是用管理员身份打开命令行, 这样创建的环境与添加的包都会安装到 `C:\ProgramData\Anaconda3` 下, 这样不会有中文路径. (注: 最开始安装 Anaconda 时安装的 base 环境就在这个目录下).