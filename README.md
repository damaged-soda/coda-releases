# Coda 发布

这里存放 Coda Mac 客户端的正式安装包、后端安装包和 Sparkle 自动更新清单 `appcast.xml`。

## Mac 客户端

- 下载：在 [Releases](https://github.com/damaged-soda/coda-releases/releases) 页面取最新的 `Coda-macOS-universal.zip`。要求 macOS 15 或更新版本。
- 安装包使用 Developer ID 签名并经 Apple 公证。
- 更新：已安装的 Coda 每天读取最新 Release 里的 `appcast.xml`，发现新版时提示安装；更新包须带有与 App 内置公钥相符的 EdDSA 签名。

## 用 npm 安装后端

可以在运行机上用 npm 安装后端，再交给自己的进程管理器常驻运行；适合容器或没有 `systemd --user` 的机器。
后端支持 Linux 和 macOS，需要 Node.js 24+、npm、Git 和 tar，并能访问 GitHub 和 npm registry。

以实际运行后端的用户执行以下命令，npm 全局安装目录需对该用户可写：

```sh
npm i -g https://github.com/damaged-soda/coda-releases/releases/latest/download/coda-backend.tgz
coda-backend run
```

`coda-backend.tgz` 是本仓库正式版 Release 的附件，携带与同版 Mac App 相同的后端。上面的命令从 GitHub 下载 npm 包。

### 启动与连接

`coda-backend run` 首次启动时安装后端和依赖，然后在前台运行，日志输出到终端。它不注册系统服务，
要在退出终端或机器重启后继续运行，需把这条命令配置到自己的进程管理器中。

- 程序和环境配置保存在 `~/.local/share/coda`，项目、会话等数据保存在 `~/.local/state/coda/data`。
- 进程管理器应使用同一个用户，并设置该用户实际的 `HOME` 和包含 Node、npm、Git 及 Agent 工具的 `PATH`。
- 停止或重启时只向主进程发送 `SIGTERM`；清理全部子进程会同时终止正在运行的 Agent。

后端运行后，在 Coda 客户端选择「通过 SSH」连接这台机器，SSH 登录账号应与运行后端的账号一致。
后端安装不会完成 Agent 认证，仍需按所用 Agent 的要求完成登录或 API 凭据配置。
客户端可以连接 npm 部署的后端，但不会替它更新或卸载。

### 更新

重新执行安装命令，再通过自己的进程管理器重启 `coda-backend run`：

```sh
npm i -g https://github.com/damaged-soda/coda-releases/releases/latest/download/coda-backend.tgz
```

下次启动时会安装新版本，保留已有数据和环境配置。
