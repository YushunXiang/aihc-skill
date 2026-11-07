# 开发机

## 创建与查询

先确定池、队列、池类型、镜像、CPU/内存/GPU、存储挂载及用户需要的访问方式。开发机的资源类型参数不能照搬 `pool list`。

```bash
aihc dev create --help
aihc dev template -f yaml -p "$WORK_DIR"
# 修改生成的 dev_conf.yaml，再提交
aihc dev create -f "$WORK_DIR/dev_conf.yaml"
aihc dev list --help
aihc dev get "$DEV_ID"
aihc dev events "$DEV_ID"
```

模板生成目录应是本次任务的工作目录，不覆盖用户已有配置。填写当前 CLI 模板所需字段；不要将官方样例里的资源 ID、镜像或挂载地址当成用户环境。

## 生命周期与更新

| 意图 | 命令入口 |
| --- | --- |
| 开机 / 关机 | `aihc dev start` / `aihc dev stop` |
| 定时停止 | `aihc dev timed-stop` |
| 导出配置 | `aihc dev export` |
| 修改配置 | `aihc dev update` |
| 制作镜像 | `aihc dev commit` |
| 查询镜像制作任务 | `aihc dev img-list` |
| 删除开发机 | `aihc dev delete` |

对目标子命令先检查帮助，再传开发机 ID 和所需参数。更新前导出配置到仓库外，在副本中做最小修改，核实该配置可用于更新，再执行 `update`。定时停止需明确延迟或启用状态，不默认替用户设置关机计划。

制作镜像需要核实目标镜像仓库等参数；创建镜像任务后记录任务 ID，用 `img-list` 查询完成状态，不能把“镜像制作已提交”说成“镜像可用”。

## 排障与完成标准

先读详情和事件，再根据实际错误检查镜像、存储、资源和网络。删除或重建开发机不属于默认排障操作。涉及系统盘或持久化路径时，先核实具体配置，不承诺关机、删除或镜像制作会保留所有数据。

创建后查询至用户要求的状态；启动/停止/更新后读取详情确认；用户要求可访问时再验证相应访问方式。不要擅自打开公网。删除需核实 ID，并以目标明确不存在作为证据。

来源（核验 2026-11-07）：[CLI 使用：开发机相关操作](https://cloud.baidu.com/doc/AIHC/s/Tm7x702fo)。
