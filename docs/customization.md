# Fork 定制说明

本文记录个人 fork 相对课程作者仓库的稳定边界。它描述“为什么这样改”和“如何继续维护”，不替代具体任务的验收记录。

## 仓库关系

| 名称 | 地址 | 职责 |
|---|---|---|
| `origin` | `seasnakes/web3-quant-sandbox` | 个人 fork 与稳定定制版本 |
| `upstream` | `congde/web3-quant-sandbox` | 课程作者的公开项目 |

当前已审阅的上游基线为 `a6cc399d715f540110ed1aea4eb3bbe536a2f29b`。后续每次同步都应更新维护记录，而不是只写“已拉取最新代码”。

## 定制原则

1. `main` 是个人 fork 的稳定分支；功能修改从 `feature/*` 通过 PR 合入。
2. 作者更新从 `automation/upstream-sync` 或 `chore/sync-upstream-*` 通过 PR 合入。
3. 优先新增配置、适配器和独立模块，尽量减少对上游核心文件的长期侵入式修改。
4. 已发布的 `main` 不重写历史，不使用 `--force` 与上游对齐。
5. 源码、课程讲稿、生成数据和真实凭证分别管理，不混在一个提交中。

## 公开与私有上下文

- `README.md` 面向使用者，描述可运行能力和命令。
- `AGENTS.md` 保存代理必须遵循的稳定目录规则、风险边界和验收命令。
- `docs/maintenance/` 保存可复用运维流程。
- `docs/templates/` 保存单任务和课程映射模板。
- `docs/v2/` 与 `book/` 是本地私有课程材料，已被 Git 忽略；公开代码不能依赖它们。
- `.env` 保存本机配置和密钥，不能提交；只在 `.env.example` 公开变量说明。

如果课程讲稿需要跨设备同步，应使用独立的私有仓库或受控知识库。公共 fork 只保存章节、代码入口、上游提交和验收命令之间的映射，不复制无授权的课程原文。

## 数据提交边界

`data/dashboard/snapshots/*.json` 的最新快照按仓库现有策略可提交，以支持离线运行；`snapshots/history/` 和运行时状态不提交。刷新快照前后必须核对：

- `origin`、`provider` 与 `updated_at`；
- `complete`、记录数量和缺失原因；
- 是否包含凭证、账户信息或不可公开数据；
- 对应页面/API 是否仍能读取；
- 快照变更是否与源码变更拆分提交。

## 版本记录

个人稳定版本建议使用能标识上游基线的标签，例如：

```bash
git tag custom-v0.1.0-upstream-a6cc399
git push origin custom-v0.1.0-upstream-a6cc399
```

标签只表示代码版本，不代表实时数据、策略收益或交易能力已经验证。
