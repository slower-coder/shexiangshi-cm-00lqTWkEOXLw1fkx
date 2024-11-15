
## 前言


最近开始整理笔记里的库存草稿，本文是 23 年 5 月创建的了（因为中途转移到 onedrive，可能还不止）


网页调起电脑程序是经常用到的场景，比如百度网盘下载，加入 QQ 群之类的


我之前做了个管理电影的项目部署在 NAS 上自己用，就需要实现在网页上一键调用电脑上的 Potplayer 播放电影，这时候直接掏出 C\# 写一个客户端就非常方便了


## 注册表操作


在 Windows 上实现就是通过注册表，将 Scheme 和对应的程序添加进去。其他系统暂时没需要就还没研究，估计也是类似的。


需要配置一下 `SchemePrefix` ，本文例子中是 demo


在网页上使用 `demo://` 开头的链接就可以唤起本机的程序了\~



```
using System.Diagnostics;
using System.Web;
using Microsoft.Win32;

const string AppName = "DemoApp";
const string SchemePrefix = "demo";

// 初始化注册表
void InitReg() {
    if (!OperatingSystem.IsWindows()) return;

    var path1 = AppName;
    var path2 = $@"{path1}\shell\open\command";

    // 设置协议名称
    var key1 = Registry.ClassesRoot.OpenSubKey(path1, true);
    if (key1 == null) {
        key1 = Registry.ClassesRoot.CreateSubKey(path1);
    }

    key1.SetValue("URL Protocol", "");
    key1.SetValue(null, $"URL:{SchemePrefix}");

    var key2 = Registry.ClassesRoot.OpenSubKey(path2, true);
    if (key2 == null) {
        key2 = Registry.ClassesRoot.CreateSubKey(path2);
    }

    var exePath = Environment.ProcessPath ?? "";

    key2.SetValue(null, $"\"{exePath}\" \"%1\"");
}

```

## 参数解析


因为是随手写的小工具，我也没有用命令行解析的库


如果用第三方库代码会更优雅


这里就做了两个命令，一个 install 另一个 open


手动执行 install 会在注册表里添加配置，之后这个程序文件就不要移动了，后续网页调起需要执行这个程序。


open 命令是网页调起时执行的，注意命令参数里的字符需要 URL 转义。



```
string action = "", value = "";
string[] cmdArgs = Environment.GetCommandLineArgs();
if (cmdArgs.Length > 1) {
    var arg = cmdArgs[1];
    Console.WriteLine($"cmd args: {arg}");

    if (arg.StartsWith($"{SchemePrefix}://")) {
        arg = arg.Replace($"{SchemePrefix}://", "");
    }

    if (arg.EndsWith("/")) {
        arg = arg.Substring(0, arg.Length - 1);
    }

    var split = arg.Split("//");
    action = split[0];
    value = split.Length > 1 ? split[1] : "";
    Console.WriteLine($"action: {action}, value: {value}");
}

switch (action) {
    case "install":
        Console.WriteLine("init reg...");
        InitReg();
        Console.WriteLine("init reg finished.");
        break;
    case "open":
        var path = HttpUtility.UrlDecode(value);
        Console.WriteLine($"open file/dir: {path}");
        if (OperatingSystem.IsWindows())
            Process.Start($"C:\\Windows\\explorer.exe", path);
        if (OperatingSystem.IsLinux())
            Process.Start("xdg-open", path);
        break;
    default:
        Console.WriteLine("不知道做啥~");
        break;
}

```

## 参考资料


* 如何在网页上打开本地应用 \- [https://segmentfault.com/a/1190000040237895](https://github.com)
* C\#进行注册表和键值操作 \- [https://zhuanlan.zhihu.com/p/403162888](https://github.com):[westworld加速](https://tianchuang88.com)


