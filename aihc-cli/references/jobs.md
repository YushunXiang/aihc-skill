# 训练任务

## 创建

先确定配置、池和队列，再确定镜像、训练入口、框架、代码位置、数据挂载、GPU 类型与数量、副本数和输出保存位置。可从已有任务或项目配置获取的内容先查；不能替用户猜测资源规模。

```bash
aihc job create --help
aihc job create -f "$JOB_FILE" -p "$POOL_ID" -q "$QUEUE"
aihc job list -p "$POOL_ID" -q "$QUEUE" --name "$JOB_NAME"
aihc job get "$JOB_ID" -p "$POOL_ID" -q "$QUEUE"
aihc job get "$JOB_ID" -p "$POOL_ID" -q "$QUEUE" --status
```

所有变量都需要实际值；按入口规则选择配置。配置文件支持 JSON/YAML。已有任务可通过 `aihc job export "$JOB_ID"` 导出后修改；新任务依据当前官方创建任务 schema 或已核验的项目模板准备，不假设存在 `job template` 命令。导出文件可能含环境变量秘密，应保存在仓库外并在分享前脱敏。

直接传参适合简单任务：核心参数包括 `--name`、`--image`、`--framework`、`--command`、`--replicas`、`--gpu`。入口命令与 `--script-file` 二选一。`--local-code` 会上传本地代码，提交前确认上传目录只包含任务所需文件；容器路径与本机路径不能混用。

参数覆盖配置文件时核对最终生效值，尤其是名称、队列、镜像、资源和入口。不要为普通训练默认启用特权、主机网络、RDMA 或容错。Ray 的 Head/Worker 资源分别配置，不能直接套用 PyTorch 的角色语义；遇到 Ray 特有参数时查当前帮助和官方对应章节。

批量提交支持重复 `-f`，任务独立提交，无联动关系。提交前列明任务数及各自资源；部分成功时只处理已确认失败项，保留未知项待查询。

## 状态、实例和日志

```bash
aihc pod list "$JOB_ID" -p "$POOL_ID" -q "$QUEUE"
aihc job logs --help
aihc job logs "$JOB_ID" --podname "$POD_NAME" -p "$POOL_ID" -q "$QUEUE"
aihc job exec --help
```

实例列表命令是 `aihc pod list`。从实际实例列表选择 Pod；不要从任务名称猜 Pod 名。连接任务实例的参数顺序和终端选项按 `job exec --help` 核实。

排队时先看任务状态、队列及资源约束；失败时看各角色状态和对应日志，区分拉镜像、数据挂载、调度、进程退出等原因。日志只证明观测到的现象，不自动证明根因。不要把进入容器执行修复当作只读检查。

## 停止、删除与验证

```bash
aihc job stop "$JOB_ID" -p "$POOL_ID" -q "$QUEUE"
aihc job delete "$JOB_ID" -p "$POOL_ID" -q "$QUEUE"
```

执行前通过详情核对 ID 和名称，操作限于用户授权的任务。停止与删除不可互换；不要为了停止计算直接删除记录。执行后查询状态；删除后正确范围内的明确不存在响应可用于验证，认证或网络错误不能作为删除成功证据。

提交完成：返回任务 ID 和当前状态。用户要求确认启动时，查询到运行状态；要求等待训练完成时，直到成功/失败终态或约定截止时间，并检查输出位置。不要将 `Created` 解释为训练成功。

来源（核验 2026-11-07）：[CLI 使用：任务相关操作](https://cloud.baidu.com/doc/AIHC/s/Tm7x702fo)。创建配置的完整 schema 通过该页链接的“API参考(v1)/训练任务相关接口/创建训练任务”核对。
