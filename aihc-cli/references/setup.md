# 安装与配置

用于 CLI 不可用、初始化配置、地域切换、认证或配置优先级排查。

## 安装

先检查已有可执行文件和 `aihc --help`。需要安装时识别操作系统与架构，从官方安装页选择对应二进制；使用用户指定版本，未指定时核对安装页当前版本。不要自动替换已安装版本。

核验时官方提供 v1.6.5 的 linux-amd64、linux-arm64、darwin-amd64、darwin-arm64 和 windows-amd64。下载路径格式为 `https://aihc-public.bj.bcebos.com/cli/release/v1.6.5/<平台>/aihc`，Windows 文件名为 `aihc.exe`。这是版本快照，不是永久的最新版地址。

Linux/macOS 下载到临时文件，确认下载成功后赋予执行权限，放到用户指定或用户级 bin 目录，并确保该目录在 PATH 中。Windows 将所在目录加入用户 PATH。安装后运行 `aihc --help` 和帮助列出的版本命令。不要仅凭文件存在判断安装成功。

## 配置选择

读取顺序（命中即停止）：

1. `-C/--config` 指定的文件。
2. `aihc config use` 选择的活动配置，管理记录位于 `~/.aihc/config_manager.yaml`。
3. 当前工作目录的 `./.aihc/config`。
4. `~/.aihc/config`。

需要隔离单次操作时优先显式 `-C`，不要隐式切换账号。只检查必要的非敏感字段和凭证是否存在，不打印整个配置。`config get` 的输出也可能包含凭证，不直接展示。

配置包含 `region`、`credentials.accesskey`、`credentials.secretkey`；可选 `defaultpool`、`defaultqueue`、`aihcusername`。`endpoint`、`useragent`、`retry` 由 CLI 生成。`aihcusername` 仅 bhcmp 平台需要。托管池默认队列存 ID，自运维池存名称。

常用帮助入口：

```bash
aihc config --help
aihc config create --help
aihc config set --help
aihc config use --help
aihc pool set --help
```

让用户通过本地可信输入方式配置密钥，避免在聊天、命令历史或日志中记录明文。`aihc config` 支持交互配置；`--path` 可指定写入位置，实际用法按当前帮助核实。不要混淆写入位置 `--path` 与读取配置 `-C`。

地域快照包括 cn-bj、cn-gz、cn-su、cn-bd、cn-fwh、cn-yq；以当前文档和用户目标为准，不能把北京作为所有任务的默认地域。

## 验证与排障

配置完成后，在用户目标地域执行必要的资源只读查询。认证失败检查所选配置、凭证有效性；权限失败检查账号在百舸中的资源权限；查询为空检查地域、池类型、队列与分页。不要通过覆盖配置或扩大权限来试错。

成功标准：CLI 可运行、有效配置上下文明确，目标范围的只读请求有可解释结果。能运行帮助不代表认证成功。

来源（核验 2026-11-07）：[安装](https://cloud.baidu.com/doc/AIHC/s/em7x6wb9v)（v1.6.5）；[配置](https://cloud.baidu.com/doc/AIHC/s/xm7x6yakv)（页面标注 v1.6.3）。
