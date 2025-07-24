# 使用 Nativefier 打包 URL 为可执行程序

Nativefier 是一个命令行工具，可以非常轻松地将任何网站包装成一个桌面应用程序（内部使用 Electron，也就是包含了 Chromium 内核）。

## 安装 node.js

1. 访问 [node.js 官网](https://nodejs.org/zh-cn/) 下载 [独立文件(.zip)](https://nodejs.org/dist/v22.17.1/node-v22.17.1-win-x64.zip) 并解压

2. 设置环境变量

    添加`NODE_HOME`，变量值为 `Node安装目录`（如 `D:\Apps\node-v22.17.1-win-x64`）

    在`Path`中添加：

    ```cmd
    %NODE_HOME%
    %NODE_HOME%\node_global
    ```

3. 自定义全局模块和缓存路径

    安装目录下创建`node_global` `node_cache`目录，并运行以下命令（替换为实际安装位置，**注意使用绝对路径**）：

    ```cmd
    npm config set prefix "D:\Apps\node-v22.17.1-win-x64\node_global"
    npm config set cache "D:\Apps\node-v22.17.1-win-x64\node_cache"
    ```

4. 验证安装是否成功：

    执行`npm -v`查看`npm`版本

    执行`npm config ls`查看`npm`配置信息

    `PowerShell`执行命令如果出现：

    > npm : 无法加载文件 D:\Apps\node-v22.17.1-win-x64\npm.ps1，因为在此系统上禁止运行脚本。有关详细信息，请参阅 https:/go.mi
    > crosoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。

    以管理员身份打开`PowerShell`，并执行

    ```powershell
    Set-ExecutionPolicy RemoteSigned -Scope LocalMachine
    ```

    可使用以下指令验证修改是否生效：

    ```powershell
    Get-ExecutionPolicy  # 应返回 RemoteSigned
    ```

5. 添加国内镜像源：可以使用阿里的国内镜像进行加速。

    ```cmd
    npm config set registry https://registry.npmmirror.com/
    ```

## 安装和使用 Nativefier

### 安装 Nativefier

打开你的命令行工具（在 Windows 上是 CMD 或 PowerShell），输入以下命令并回车，全局安装 Nativefier：

```cmd
npm install -g nativefier
```

### 封装 URL

继续在命令行中，使用以下命令即可。把 <https://cn.bing.com> 换成你的目标 URL。

```cmd
nativefier "https://cn.bing.com"
```

命令执行完毕后，会在你当前目录下生成一个以网站名命名的文件夹，里面就包含了 .exe 文件。

### 添加更多自定义选项

- **命名应用**: -n "My Awesome App"
- **指定图标**: -i "C:\path\to\my\icon.ico"
- **指定 Electron 版本 (从而控制 Chromium 版本)**: --electron-version "25.0.0" (你可以在 [Electron Releases](https://releases.electronjs.org/release) 找到 Electron 版本)
- **单实例运行** (防止打开多个窗口): --single-instance
- **全屏启动**: --full-screen
- **隐藏窗口边框**: --frame false

例如：

```cmd
nativefier "https://chat.openai.com" -n "My ChatGPT" -i "D:\icons\chatgpt.ico" --electron-version "28.1.0" --single-instance
```

## 生成 URL 可动态配置的应用

在目录下包含以下文件：

```text
.
├── config.json
├── favicon.ico
├── index.html
└── preload.js
```

`config.json`

```json
{
    "url": "http://localhost:8080/"
}
```

`preload.js`

```javascript
const { contextBridge } = require("electron");
const fs = require("fs");
const path = require("path");

contextBridge.exposeInMainWorld("myAPI", {
    loadConfig: () => {
        try {
            const configPath = path.join(__dirname, "config.json");
            const rawData = fs.readFileSync(configPath, "utf8");
            return JSON.parse(rawData);
        } catch (e) {
            console.error("Preload script failed to load config.json:", e);
            return { error: e.message };
        }
    },
});
```

`index.html`

```html
<!DOCTYPE html>
<html>
    <head>
        <title>正在加载...</title>
        <meta charset="UTF-M" />
        <style>
            body {
                font-family: sans-serif;
                display: flex;
                justify-content: center;
                align-items: center;
                height: 100vh;
                margin: 0;
                background-color: #f0f0f0;
            }
            .loader {
                font-size: 1.2em;
                color: #555;
            }
        </style>
    </head>
    <body>
        <div class="loader">正在加载应用，请稍候...</div>
        <script>
            document.addEventListener("DOMContentLoaded", () => {
                const config = window.myAPI.loadConfig();
                if (config && config.url) {
                    window.location.href = config.url;
                } else {
                    document.body.innerHTML =
                        "错误: 无法加载配置。<br>详情: " +
                        (config.error || "未知错误");
                }
            });
        </script>
    </body>
</html>
```

使用以下命令即可实现生成 URL 可动态配置的应用（**在`cmd`中执行**）：

```cmd
nativefier "file:./index.html" --name "App" --icon "favicon.ico" --inject "./preload.js" --browserwindow-options "{\"webPreferences\":{\"contextIsolation\":true}}" --internal-urls ".*"
```

这条命令：

1. **打包 `index.html`**：打包本地启动器 (`index.html`)，为动态配置提供了基础。
2. **注入 `preload.js`**：解决了读取本地 `config.json` 的跨域问题。
3. **开启 `contextIsolation`**：让 `preload.js` 的安全桥梁 (`contextBridge`) 能够工作。
4. **定义 `internal-urls`**：确保了从本地启动器跳转到任何一个网络 URL 时，导航行为是正确的（在应用内打开）。

具体地：

- `nativefier "file:./index.html"`
    这是命令的核心目标。它告诉 Nativefier，你要封装的不是一个在线的 URL ，而是一个本地的 HTML 文件。这是实现“通过配置文件动态跳转”方案的关键第一步。我们不直接打包最终的目标网址，而是打包一个本地的“启动器”页面。这个启动器页面再通过 JavaScript 去读取配置并执行跳转。`file:./index.html`指定协议为本地文件，并使用相对路径。

- `--name "App"` 和 `--icon "favicon.ico"`

    这两个选项分别设置了应用的名称和图标。

- `--inject "./preload.js"`

    将指定的 `preload.js` 脚本注入到应用中。这是解决“本地文件跨域读取 `config.json`”问题的**正确且安全**的方案。这个 `preload` 脚本运行在一个有特权的环境里，它可以使用 Node.js 的 `fs`模块来读取本地的 `config.json`，然后通过 `contextBridge` 安全地将配置信息暴露给 `index.html` 页面。

- `--browserwindow-options "{\"webPreferences\":{\"contextIsolation\":true}}"`

    设置 Electron 浏览器窗口的选项。这是让 `preload.js` 能够正常工作的**关键**。

    `contextIsolation: true` 开启了“上下文隔离”安全模式。在这个模式下，`preload.js` 和 `index.html` 的 JavaScript 环境是隔离的，防止网页端的恶意代码影响到有更高权限的 Node.js 环境。`contextBridge` API 必须在 `contextIsolation: true` 的情况下才能使用。如未设置，则会遇到 `contextBridge API can only be used when contextIsolation is enabled`错误。

- `--internal-urls ".*"`

    定义哪些 URL 被视为“内部链接”。当用户点击一个被判定为内部的链接时，它会在当前应用窗口内打开；如果链接不匹配这个规则，它会被传递给系统默认的浏览器打开。

    `".*"` 是一个正则表达式，意思是“匹配任何字符（.）零次或多次（\*）”，简单来说就是**匹配所有 URL**。

    当你的 `index.html` 执行 `window.location.href = config.url;` 时，Nativefier 需要决定这个新的 URL 是在应用内打开，还是在外部浏览器打开。默认情况下，如果这个 URL 的域名和你最初打包的 URL（这里是 `file://` 协议）不一致，它可能会被当作“外部链接”。

    通过设置 `--internal-urls ".*"`，你强制告诉 Nativefier：“**无论 `config.json` 里配置的是什么网址，都把它当作是我自己应用的内部页面，请直接在当前窗口加载它。**”这解决了从 `file://` 协议跳转到 `http://` 或 `https://` 协议时可能出现的导航问题，从而避免了白屏。
