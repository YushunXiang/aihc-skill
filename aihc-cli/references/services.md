# 推理服务

## 部署

明确镜像、启动命令、资源池与队列、实例规格和数量、模型存储路径、容器端口及健康检查。公网访问仅在用户需求包含它时配置。

```bash
aihc service create --help
aihc service template -f yaml -p "$WORK_DIR"
# 填写生成的 service_conf.yaml 后创建
aihc service create -f "$WORK_DIR/service_conf.yaml"
aihc service list --help
aihc service get "$SERVICE_ID"
aihc service pods "$SERVICE_ID"
```

使用独立工作目录生成模板，避免覆盖。模型路径必须是容器可访问的挂载路径。创建成功后记录 ID，查询实例和服务状态；用户要求服务可用时，结合健康状态验证实际端点，不把资源创建成功当成推理成功。

## 更新与运维

| 需求 | 命令入口及要点 |
| --- | --- |
| 更新 | `service export` 导出，修改副本，再 `service update` |
| 扩缩容 | `service scale`，核实 `--instance-count` |
| 公网配置 | `service net-config`，明确开关和目标 EIP |
| 重启 | `service restart`，区分服务与指定实例 |
| 实例组 | `service pod-groups` |
| 删除 Pod 触发重建 | `service pod-delete`，属于变更操作 |
| 摘流 / 恢复 | `service disable-service-pod`，核实 `--block` |
| 进入容器 | `service exec`，确认实例和目标命令 |
| 日志 | `service logs`，按帮助选择实例、时间范围和跟踪方式 |
| 变更记录 | `service changelog` |
| 删除 | `service delete` |

表内命令均以 `aihc` 为前缀，执行前查具体子命令帮助。不要推断不同命令的 `-i`、`-p`、`-f` 含义相同。`service get -o` 是输出格式，`service export -o` 是输出文件路径。

导出文件保存在仓库外并检查敏感字段。更新只修改用户要求的配置。操作影响范围明确且已获授权时执行；仅要求排查故障时先读日志和变更记录，不自动重启或删除 Pod。

## 验证

扩缩容检查期望数量和实际就绪数量；更新检查变更记录及实例就绪状态；重启检查新实例状态。摘流、网络修改按对应详情核实。持续失败时返回观测证据，不无限重试或自动扩大资源。

来源（核验 2026-11-07）：[CLI 使用：推理服务相关操作](https://cloud.baidu.com/doc/AIHC/s/Tm7x702fo)。
