# Coda 发布

这里存放 Coda Mac 客户端的正式安装包、后端安装包和 Sparkle 自动更新清单 `appcast.xml`。

## Mac 客户端

- 下载：在 [Releases](https://github.com/damaged-soda/coda-releases/releases) 页面取最新的 `Coda-macOS-universal.zip`。要求 macOS 15 或更新版本。
- 安装包使用 Developer ID 签名并经 Apple 公证。
- 更新：已安装的 Coda 每天读取最新 Release 里的 `appcast.xml`，发现新版时提示安装；更新包须带有与 App 内置公钥相符的 EdDSA 签名。

## 用 npm 安装后端

可以在运行机上用 npm 安装后端，再交给自己的进程管理器常驻运行；适合容器或没有 `systemd --user` 的机器。
后端支持 Linux 和 macOS，需要 Node.js 24+、npm、Git 和 tar，并能访问 GitHub（包括 api.github.com）和 npm registry。

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

### 自动更新

`coda-backend run` 默认自动更新：启动一分钟后检查最新正式版，此后每六小时检查一次。
发现新版后下载并核对安装包的长度和 SHA-256，准备好后端及依赖后再切换，保留已有数据和环境配置。
自动更新只写入用户目录，不修改全局 npm 安装，也不升级 Node.js。

下载或依赖准备失败时，当前后端继续运行，稍后重试。新版本未通过健康检查时，只要旧版本仍能打开当前数据库，就回到旧版本；
如果新版本已经升级了数据库、旧版无法打开，则保留新版本并在日志里说明原因。已尝试启动但失败的版本不反复自动安装，等待下一次发布。
切换期间客户端会短暂断开，正在运行的 Agent 会话保留。

需要固定当前版本时，使用以下常驻命令：

```sh
coda-backend run --no-auto-update
```

也可以在进程管理器中设置环境变量 `CODA_NPM_AUTO_UPDATE=0`。关闭自动更新会保留已经安装的版本。

旧版 npm 启动器首次启用自动更新，需要手动安装支持此能力的版本并重启一次；之后由启动器自动更新。
要手动安装新版本，也可以执行以下命令，再通过自己的进程管理器重启 `coda-backend run`：

```sh
npm i -g https://github.com/damaged-soda/coda-releases/releases/latest/download/coda-backend.tgz
```

启动时会保留已更新的后端，旧的全局 npm 包不会将它降级。`coda-backend --version` 可以查看已安装的后端版本。
