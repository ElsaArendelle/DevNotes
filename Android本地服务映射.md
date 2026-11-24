# Android本地服务映射

> 用途：把电脑上正在开发的网站直接放到手机的真实环境里测试，不用先部署到线上。

## 一、原理

手机打开 USB 调试后，系统里会常驻一个调试守护进程，电脑上的 adb 程序通过数据线和它建立一条通道。

核心命令是 `adb reverse`：它让手机在自己的本地回环地址上开一个端口，手机上任何应用访问 `localhost:这个端口`，数据都会经数据线送到电脑，再转给电脑本机的同名端口。

对手机浏览器来说，它访问的是“本机”，算安全来源。所以 Service Worker、网页推送这些平时只有 HTTPS 才能用的功能，在本地版上也能测。

另一种做法是手机和电脑连同一个 Wi-Fi，手机直接访问电脑的局域网 IP。但那样不是安全来源，这些功能用不了，还会碰到防火墙弹窗，所以不如 `adb reverse`。

## 二、准备

**手机**

1. 设置 → 关于手机 → 连点版本号 7 次，打开开发者选项。
2. 开发者选项里打开“USB 调试”。
3. 插上数据线后，手机会弹“允许 USB 调试吗？”，点允许。

**电脑**

电脑上已经有 adb，位置是：

```
%AppData\Local\Android\Sdk\platform-tools\adb.exe
```

它所在的文件夹不在 PATH 环境变量里，所以直接敲 `adb` 会报“无法将 adb 项识别为 cmdlet”。

## 三、让 adb 命令能用

**临时用一次（写完整路径）**

```powershell
& "C:\Users\25705\AppData\Local\Android\Sdk\platform-tools\adb.exe" devices
```

**只对当前这个 PowerShell 窗口生效**

```powershell
$env:Path += ";C:\Users\25705\AppData\Local\Android\Sdk\platform-tools"
```

关掉窗口就失效。

**永久生效（推荐，只执行一次）**

```powershell
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", "User") + ";C:\Users\25705\AppData\Local\Android\Sdk\platform-tools", "User")
```

执行完要关掉 PowerShell 再重新打开。

不想敲命令也可以用界面：Win 键搜索“编辑账户的环境变量” → 用户变量里的 `Path` → 编辑 → 新建 → 粘贴上面那个文件夹路径 → 确定。同样要重开 PowerShell。

**确认手机已连接**

```powershell
adb devices
```

正常输出：

```
List of devices attached
QMFDU20429022039       device
```

- 显示 `device`：正常。
- 显示 `unauthorized`：手机上还没点“允许”。
- 列表是空的：线没插好，或者手机只选了“充电”模式，改成“传输文件”。

## 四、建立映射

1. 电脑上先把本地服务起好，比如监听 8080 的前端开发服务。
2. 建立映射：

   ```powershell
   adb reverse tcp:8080 tcp:8080
   ```

   前面是手机的端口，后面是电脑的端口，可以不一样，比如 `adb reverse tcp:8080 tcp:3000`。

3. 手机浏览器打开 `http://localhost:8080/`，看到的就是电脑上跑的那一份。

以「一起过」为例：电脑起后端 8000 和前端 8080，只需要映射 8080，因为前端开发服务会把 `/api` 转给电脑上的 8000。手机浏览器打开 `http://localhost:8080/prototype/index.html`。

**查看和撤销**

```powershell
adb reverse --list
adb reverse --remove tcp:8080
adb reverse --remove-all
```

## 五、adb 的三个常用方向

| 命令          | 方向             | 用途                           |
| ------------- | ---------------- | ------------------------------ |
| `adb reverse` | 手机 → 电脑      | 手机访问电脑上的服务           |
| `adb forward` | 电脑 → 手机      | 电脑访问手机里的端口或内部通道 |
| `adb shell`   | 在手机上执行命令 | 查系统信息、看哪个应用在前台   |

记法：`forward` 是电脑访问手机里的东西，`reverse` 是手机访问电脑里的东西。

`adb shell` 的例子：

```powershell
adb shell getprop ro.product.model
adb shell dumpsys window | findstr mCurrentFocus
```

## 六、读取手机页面里的运行状态（进阶）

手机浏览器的内核有一条调试通道，电脑连上去可以读页面里运行的代码。

1. 找到通道名：

   ```powershell
   adb shell cat /proc/net/unix | findstr devtools
   ```

   会列出类似 `@huawei_webview_devtools_remote_19159` 的名字。开头的 `@` 不要带。

2. 把电脑的端口通到这个通道：

   ```powershell
   adb forward tcp:9421 localabstract:huawei_webview_devtools_remote_19159
   ```

3. 电脑浏览器打开 `http://127.0.0.1:9421/json`，就能看到手机上打开的页面列表。

4. 用完撤销：

   ```powershell
   adb forward --remove-all
   ```

通道名后面的数字是进程号，浏览器重启后会变，要重新查。

## 七、注意事项

- 拔线、手机重启、adb 服务重启后，映射都会消失，要重新执行 `adb reverse`。
- 电脑上的服务要能从本机回环地址访问到。如果它只绑定了某个局域网 IP，转过去会连不上。
- 页面在后台被挂起时，带网络请求的读取会超时，只读不联网的信息才行。
- 纯血鸿蒙（HarmonyOS NEXT）上 adb 用不了，对应的工具是 hdc，反向映射的子命令是 `hdc rport`。

## 八、速查

```powershell
# 1. 确认连接
adb devices

# 2. 手机 localhost:8080 -> 电脑 8080
adb reverse tcp:8080 tcp:8080
adb reverse --list

# 3. 测完清理
adb reverse --remove-all
adb forward --remove-all
```