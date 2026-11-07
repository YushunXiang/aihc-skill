# AIHC CLI Skill

面向 AI 编程助手的百度百舸 AIHC CLI 使用技能。支持环境配置、资源池与队列查询、训练任务、开发机、推理服务及 OpenAPI 透传。

## 安装

将本仓库的 `aihc-cli/` 目录复制到支持 SKILL.md 的助手技能目录。例如 Codex：

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R aihc-cli "${CODEX_HOME:-$HOME/.codex}/skills/aihc-cli"
```

若目标已存在，先检查已有内容，避免嵌套复制或覆盖个人修改。使用前按百度官方文档安装 `aihc` 并在本地配置账号；不向聊天或仓库提交 AK/SK。

## 使用示例

- “用 $aihc-cli 查询这个资源池下的训练任务。”
- “把当前项目提交到指定百舸队列训练，并检查是否启动。”
- “查看这个开发机的状态和最近事件。”
- “排查这个推理服务不就绪的原因，先查看日志。”

技能入口：[aihc-cli/SKILL.md](aihc-cli/SKILL.md)。六个参考文件按需加载；具体命令以安装版本的帮助为准。该项目不包含凭证、运行时导出配置或对 CLI 的二次封装。

## 文档与验证范围

基于百度官方[概览](https://cloud.baidu.com/doc/AIHC/s/Fm7x6ox2x)、[安装](https://cloud.baidu.com/doc/AIHC/s/em7x6wb9v)、[配置](https://cloud.baidu.com/doc/AIHC/s/xm7x6yakv)和[CLI 使用](https://cloud.baidu.com/doc/AIHC/s/Tm7x702fo)文档整理，核验日期为 2026-11-07。安装文档版本为 v1.6.5，配置文档标注 v1.6.3。

发布前检查 Skill 格式、内部链接及本机 CLI 帮助中的命令入口。未进行云端资源创建、停止、删除或推理端点测试；这些需要实际账号、资源和对应操作授权。
