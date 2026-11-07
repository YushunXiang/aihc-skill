# 资源池与队列

用于定位操作范围、查询资源和解决“列表为空”。查询前遵循入口的配置选择规则。

## 查询流程

1. 从用户目标判断全托管或自运维资源池；不确定时分别查询，不随意选择第一个池。
2. 查询池详情，再列出该池队列。记录地域、池 ID、池类型、队列名称及 ID。
3. 根据用户已有选择或可查询到的镜像、资源需求匹配目标；仍有多个合理候选时询问。

```bash
aihc pool list --help
aihc pool list --resourcePoolType dedicatedV2
aihc pool list --resourcePoolType common
aihc pool get -p "$POOL_ID"
aihc queue list -p "$POOL_ID"
aihc queue get "$QUEUE_ID" -p "$POOL_ID"
```

示例变量必须由实际查询或用户输入赋值；使用既定配置时追加 `-C "$AIHC_CONFIG"`。

## 易错点

- `pool list` 的文档默认类型是 `common`；只查询默认类型可能漏掉全托管池。
- 训练任务的 `-q`：全托管池使用队列 ID，自运维池使用队列名称。不要仅按参数的“队列ID”说明判断。
- `pool list`、`queue list`、`job list` 的分页和排序参数拼写不同，逐个查看帮助。
- 不将 `pool list` 的类型值直接用于开发机或服务的 `--resource-pool-type`；后者根据各自帮助和模板确定。
- GPU 总数或瞬时空闲数不等于保证可调度容量；还可能受队列配额、调度规则与权限影响。

成功标准：目标地域、池与队列唯一明确，用户需要的资源信息附带查询范围。只有用户要求保存默认值时才使用 `aihc pool set`。

来源（核验 2026-11-07）：[CLI 使用：资源池、队列、任务查询](https://cloud.baidu.com/doc/AIHC/s/Tm7x702fo)；[配置](https://cloud.baidu.com/doc/AIHC/s/xm7x6yakv)。
