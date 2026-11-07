# 安装与配置

用于 CLI 不可用、初始化配置、地域切换、认证或配置优先级排查。

## 安装

先检查已有可执行文件和 `aihc --help`。需要安装时识别操作系统与架构，从官方安装页选择对应二进制；使用用户指定版本，未指定时核对安装页当前版本。不要自动替换已安装版本。

核验时官方提供 v1.6.5 的 linux-amd64、linux-arm64、darwin-amd64、darwin-arm64 和 windows-amd64。下载路径格式为 `https://aihc-public.bj.bcebos.com/cli/release/v1.6.5/<平台>/aihc`，Windows 文件名为 `aihc.exe`。这是版本快照，不是永久的最新版地址。

### Linux / macOS

下面是 Bash 安装示例，适用于首次安装。先运行 `command -v aihc`；如已安装，先查看版本与路径，仅在用户要求升级或替换时调整。安装目录采用用户级 `~/.local/bin`，无需 sudo。需要 `curl`，示例固定为已核验的 v1.6.5。

```bash
# 在 Bash 中执行；子 shell 中的失败不会退出用户当前终端。
(
  set -eu
  aihc_version='v1.6.5'
  case "$(uname -s)" in
    Linux) aihc_os='linux' ;;
    Darwin) aihc_os='darwin' ;;
    *) echo '此示例只支持 Linux/macOS' >&2; exit 1 ;;
  esac
  case "$(uname -m)" in
    x86_64|amd64) aihc_arch='amd64' ;;
    aarch64|arm64) aihc_arch='arm64' ;;
    *) echo '官方安装页未列出该架构，请先核实支持情况' >&2; exit 1 ;;
  esac
  aihc_bin_dir="$HOME/.local/bin"
  if [ -e "$aihc_bin_dir/aihc" ] || [ -L "$aihc_bin_dir/aihc" ]; then
    echo '目标文件已存在，请先检查已有版本' >&2
    exit 1
  fi
  aihc_tmp_dir="$(mktemp -d)"
  trap 'rm -rf "$aihc_tmp_dir"' EXIT
  curl --fail --location --show-error \
    "https://aihc-public.bj.bcebos.com/cli/release/$aihc_version/$aihc_os-$aihc_arch/aihc" \
    --output "$aihc_tmp_dir/aihc"
  chmod 755 "$aihc_tmp_dir/aihc"
  "$aihc_tmp_dir/aihc" --help >/dev/null
  "$aihc_tmp_dir/aihc" version
  mkdir -p "$aihc_bin_dir"
  mv "$aihc_tmp_dir/aihc" "$aihc_bin_dir/aihc"
)
```

安装成功后，在当前终端执行：

```bash
export PATH="$HOME/.local/bin:$PATH"
command -v aihc
aihc version
aihc --help
```

要对新终端持久生效，将上面的 `export PATH=...` 一行加入实际使用的 shell 启动文件：交互式 Bash 通常用 `~/.bashrc`，Zsh 用 `~/.zshrc`；登录 shell 需检查其 profile 是否加载对应文件。已有相同配置时不重复追加。重新打开终端后验证。CI/非交互进程可能不加载这些文件，应显式设置 PATH 或使用完整路径。

### Windows（PowerShell）

官方提供 windows-amd64；其他 Windows 架构先核实兼容性，不虚构下载地址。下面安装到用户目录，已存在文件时停止。

```powershell
$ErrorActionPreference = 'Stop'
$aihcVersion = 'v1.6.5'
$aihcBinDir = Join-Path $env:LOCALAPPDATA 'AIHC\bin'
$aihcExe = Join-Path $aihcBinDir 'aihc.exe'
if (Test-Path $aihcExe) { throw '目标文件已存在，请先检查已有版本' }
$aihcTemp = Join-Path ([IO.Path]::GetTempPath()) ([guid]::NewGuid().ToString() + '.exe')
try {
    Invoke-WebRequest -Uri "https://aihc-public.bj.bcebos.com/cli/release/$aihcVersion/windows-amd64/aihc.exe" -OutFile $aihcTemp
    & $aihcTemp --help
    if ($LASTEXITCODE -ne 0) { throw '下载后的 CLI 无法运行' }
    New-Item -ItemType Directory -Force -Path $aihcBinDir | Out-Null
    Move-Item -Path $aihcTemp -Destination $aihcExe
} finally {
    if (Test-Path $aihcTemp) { Remove-Item $aihcTemp }
}
$env:Path = "$aihcBinDir;$env:Path"
Get-Command aihc
aihc version
aihc --help
```

当前 PowerShell 的 PATH 设置不会持久化。将 `$aihcBinDir` 对应的实际目录通过“环境变量 → 用户变量 → Path → 新建”加入，重新打开终端验证；不要覆盖原有 Path。

### 常见安装问题

| 现象 | 处理 |
| --- | --- |
| `command not found` / 找不到命令 | 用完整路径运行以区分安装失败和 PATH 问题；检查启动文件与实际 shell |
| `Permission denied` | 检查目标文件的执行权限和所在挂载点是否禁止执行，不盲目 sudo |
| `Exec format error` / 无法运行此应用 | 重新核对操作系统、架构和下载是否成功，不能靠 chmod 修复架构不匹配 |
| 版本与预期不同 | 用 `type -a aihc`（Windows 用 `Get-Command aihc -All`）检查多个安装；必要时刷新 shell 命令缓存或重开终端 |
| macOS 提示无法校验 | 确认文件来自官方地址且用户要信任该文件后，可对该文件执行 `xattr -d com.apple.quarantine "$HOME/.local/bin/aihc"`；不要递归移除目录隔离属性或关闭系统防护 |
| 下载失败 / 404 | 检查网络及官方页面版本与平台链接，不能把错误页面当作二进制安装 |

安装验证只运行帮助和版本，不调用云端业务接口。若用户要求升级，先记录原版本和路径，查 `aihc upgrade --help` 或选择官方指定版本下载；不要因文档版本更高就自动升级。

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
