# 写在前面

因为最近公司的聊天工具转用了企业微信，所以想配合企业微信里的机器人搞点好玩的东西，比如定时提醒上下班，提醒干饭😁。

因为本身对launchctl一窍不通，所以在应用的时候遇到了很多奇奇怪怪的问题，包括但不限于文件权限、plist编写规范、执行命令行报错不知道怎么查等等，于是就有了这篇文章。我遇到问题的地方会重点标注。

# 正文

## **Launchctl 的介绍**

Launchctl 是 Mac 系统自带的定时任务工具，与 crontab 功能类似。

## **Launchctl 的使用**

1.  以简单的例子展示如何使用 Launchctl，比如现有一个位于 /usr/local/bin 目录的 echo.sh 脚本，脚本内容如下：

```
#!/bin/sh
echo `date`
```

并执行 `chmod a+x echo.sh` 获取执行权限。

期望每 30 秒钟以定时任务运行上面的脚本。

2.  进入 /Library/LaunchAgents 目录下：

`cd /Users/frank/Library/LaunchAgents`

3.  创建 com.echo.plist 定时任务描述文件：

`vim com.echo.plist`

输入以下内容：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- 名称，要全局唯一 -->
    <key>Label</key>
    <string>com.echo</string>
    <!-- 是否禁用 -->
    <key>Disabled</key>
    <false/>

    <!-- 脚本任务 -->
    <key>Program</key>
    <string>/usr/local/bin/echo.sh</string>
    
    <!-- 脚本之行的命令和参数 -->
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/echo.sh</string>
    </array>
    
    <!-- 轮询执行，每10s执行一次脚本 -->
    <key>StartInterval</key>
    <integer>10</integer>
    
    <!-- 首次加载时是否立即执行一次 -->
    <key>RunAtLoad</key>
    <true/>
    
    <!-- 日志输出文件 -->
    <key>StandardOutPath</key>
    <string>/usr/local/bin/echo.log</string>
    
    <!-- 错误日志输出文件 -->
    <key>StandardErrorPath</key>
    <string>/usr/local/bin/echo.err</string>
</dict>
</plist>
```

4.  .plist 文件标签说明：

-   `Label`：定时任务的名称，这里 Label 的值是 `com.echo.plist`，要全局唯一，可通过：`launchctl list` 查看已有的定时任务
-   `Program`：是要运行的脚本的名称，这里是 /usr/local/bin/echo.sh
-   `ProgramArguments`： 脚本运行的参数，第一个是运行的命令，其余是运行需要的参数，这里是 /usr/local/bin/working.sh，由于运行这个脚本不需要其他参数，所有只有这一个命令
-   `RunAtLoad`：表示加载定时任务即开始执行脚本
-   `StartInterval`： 定时任务的周期，单位为秒，这里 30 代表每 30s 执行一次脚本
-   `StandardOutPath`： 标准输出日志的路径
-   `StandardErrorPath`： 标准错误输出日志的路径

加载该定时任务：

`launchctl load -w com.echo.plist`

`-w` 参数会将 .plist 文件中无意义的键值过滤掉，建议加上。

这时，定时任务已加载，由于加上了：

```
<key>RunAtLoad</key>
<true/>
```

所以该定时任务成功加载后就开始执行，可以在 /Users/demo/run.log 看到每 10s 打印当前时间：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43151a7c515a45e593218266c5b06a75~tplv-k3u1fbpfcp-zoom-1.image)

如果我们的目标不是每 30 秒执行脚本，而是每天固定时间执行脚本，比如每天晚上 22:00 执行脚本，那么我们需要对 .plist 文件进行如下修改：

-   删除：

```
<key>StartInterval</key>
<integer>30</integer>
```

-   添加：

```
<key>StartCalendarInterval</key>
<dict>
    <key>Hour</key>
    <integer>22</integer>
    <key>Minute</key>
    <integer>0</integer>
</dict>
```

新的 .plist 文件内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- 名称，要全局唯一 -->
    <key>Label</key>
    <string>com.echo</string>
    <!-- 是否禁用 -->
    <key>Disabled</key>
    <false/>

    <!-- 脚本任务 -->
    <key>Program</key>
    <string>/usr/local/bin/echo.sh</string>
    
    <!-- 脚本之行的命令和参数 -->
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/echo.sh</string>
    </array>
    
    <!-- 22:00执行一次脚本 -->
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>22</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    
    <!-- 首次加载时是否立即执行一次 -->
    <key>RunAtLoad</key>
    <true/>
    
    <!-- 日志输出文件 -->
    <key>StandardOutPath</key>
    <string>/usr/local/bin/echo.log</string>
    
    <!-- 错误日志输出文件 -->
    <key>StandardErrorPath</key>
    <string>/usr/local/bin/echo.err</string>
</dict>
</plist>
```

`StartInterval` 标签代表每多少秒执行，`StartCalendarInterval` 标签代表指定时间点执行，所以这两个标签在同一个 .plist 文件中只能存在一个。`StartCalendarInterval` 的 key 有：

| **Key** | **Type** | **Values**                               |
| ------- | -------- | ---------------------------------------- |
| Month   | Integer  | Month of year (1..12, 1 being January)   |
| Day     | Integer  | Day of month (1..31)                     |
| Weekday | Integer  | Day of week (0..7, 0 and 7 being Sunday) |
| Hour    | Integer  | Hour of day (0..23)                      |
| Minute  | Integer  | Minute of hour (0..59)                   |

同样使用 `launchctl load -w com.echo.plist` 命令可使新的定时任务运行起来，在每天 22:00 时运行。

## 配合企微机器人创建定时提醒

1.  首先我们添加一个机器人，然后复制webhook地址

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e4f7bf053d644f0855296a52c70ff68~tplv-k3u1fbpfcp-zoom-1.image)

2.  再找到自己想要配置的内容格式，把脚本文件内容修改一下

```
#!/bin/sh
curl 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=ef1a6985' \
   -H 'Content-Type: application/json' \
   -d '
   {
        "msgtype": "text",
        "text": {
            "content": "干饭啦！干饭啦！干饭啦！\n到点儿不干饭，一天活儿白干！",
            "mentioned_mobile_list":["@all"]
        }
   }'

echo `date`
```

3.  然后我们添加一个11:56的定时任务就完成了，可以看到

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97ad5e6cad244a9bbeed305a3bf83103~tplv-k3u1fbpfcp-zoom-1.image)

4.  如果你愿意的话，还可以多添加几个，比如添加一个10:30的上班提醒，你就会看到😁

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1505ea41e5f45698396ce184a247de5~tplv-k3u1fbpfcp-zoom-1.image)

5.  企微机器人的原理其实很简单，就是提供给我们一个api，我们组织好模版内容作为参数调用就可以了。

## **Launchctl 细节**

### **.plist 文件存放位置**

上述展示中，.plist 描述文件是存放在 /Library/LaunchDaemons 目录下，除了这个目录还可以存在其他位置：

| **Type**       | **Location**                  | **Run on behalf of**                             |
| -------------- | ----------------------------- | ------------------------------------------------ |
| User Agents    | ~/Library/LaunchAgents       | Currently logged in user                         |
| Global Agents  | /Library/LaunchAgents         | Currently logged in user                         |
| Global Daemons | /Library/LaunchDaemons        | root or the user specified with the key UserName |
| System Agents  | /System/Library/LaunchAgents  | Currently logged in user                         |
| System Daemons | /System/Library/LaunchDaemons | root or the user specified with the key UserName |

存放 在 LaunchAgents 代表用户登录后启动任务；存放在 LaunchDaemons 代表用户登录前就启动任务。一般将自定义的 .plist 存放在 /Library/LaunchDaemons 目录下。

### **Launchctl 相关命令**

查看已有的任务：

`launchctl list`

查看指定的任务 xxx 是否存在（加载）：

`launchctl list | grep xxx`

加载指定的任务 xxx：

`launchctl load -w xxx`或`launchctl load xxx`

删除任务:

`launchctl unload -w xxx`

`launchctl load xxx`

开始任务：

`launchctl start xxx`

停止任务：

`launchctl stop xxx`

# 写在最后

-   `如果任务修改了，必须先 unload，然后重新执行 load`
-   `start 可以测试任务，这个命令是立即执行，不管时间到了没有`
-   `执行 start 和 unload 命令前，任务必须先执行过 load，否则报错`
-   `.plist 中不要添加 KeepAlive 标签，否则无条件 10s 定时执行`
-   `要保证 .plist 文件内容书写规范，否则报错。`
-   `要保证 .plist 文件权限是644，.sh 脚本权限755，.log文件权限644，否则报错`
-   `文件存放的位置也可能导致报错，或者执行不成功`
-   `如果不熟悉的话建议严格按照文档中的步骤先写一个最简单的案例，然后一步一步延展！`
