# dsh-memory-layer

> **给 DeepSeek Harness 的记忆层**：本地单文件、零服务、零 API，靠**触发信号**把该想起的记忆在该想起的时候拿出来。

**v0.10.0** · MIT License · DSH ≥ 0.1.1-rc.2（已在 0.2.0-rc.2 桌面版实测）· Node ≥ 22.19.0

---

## 它是什么

一句话：**只做记忆，不做记忆平台。**

- **不做**：外置服务、向量检索、嵌入模型、知识图谱、云端同步、Web UI。
- **做**：一个 SQLite 单文件 + 一个"什么时候该想起什么"的确定性规则集。

它和"记忆平台"（如外置记忆服务）的区别，不是功能多少，而是**没有额外要跑的东西**：装完就是一个插件，数据在你自己机器的 `.dsh/biomemory/biomemory.db` 里，拷走文件就是迁移。

---

## 核心功能

### 记忆存储

- **单文件 SQLite**：数据存储在本地，支持 WAL 模式
- **隔离维度**：`realm` 区分用户级和工作目录级记忆
- **记忆类型**：`workflow` / `error` / `decision` / `fact`
- **墓碑机制**：删除不是物理删除，而是标记为墓碑，可恢复

### 触发式召回

- **信号驱动**：根据当前会话的工作目录和工具调用历史触发记忆
- **触发信号**：
  - `file:<文件名>` - 文件相关操作
  - `tool:<工具名>` - 工具调用
  - `err:<错误代号>` - 错误处理
- **智能注入**：按类型优先级排序（workflow > error > decision > fact），每次最多 8 条、最多 1200 字符

### 自动清理

- **同键覆盖**：同 realm + type + key 的新记忆覆盖旧记忆
- **过期清理**：fact 30 天无使用则进墓碑，workflow/error/decision 90 天无使用权重减半
- **永不删除**：error 与 decision 类型永不自动删除
- **日志记录**：每次清理记录到 `compact.log`

### 工作流链

- **自动升级**：同一触发串出现超过 3 次，相关记忆自动升级为 workflow
- **链式记录**：按时间顺序拼成工作流链

---

## 安装

```bash
# 需要已安装 git；--profile 换成你的 profile 名
dsh plugin --profile web add github:KLRSL/dsh-memory-layer

# 本地开发（link）
dsh plugin --profile web add link:./dsh-memory-layer
```

装完重启 DSH（桌面版会热挂载）。

---

## 使用方法

### 工具 `memory`

| action | 说明 |
|--------|------|
| `add` | 写入一条记忆 |
| `query` | 关键词查询 |
| `list` | 浏览记忆 |
| `update` | 修改记忆 |
| `remove` | 删除记忆（墓碑） |
| `restore` | 恢复误删 |
| `pin` / `unpin` | 锁定/解锁记忆 |
| `compact` | 清理记忆（支持 dry-run） |
| `dump` | 导出 Markdown |

### 命令 `/memory`

```
/memory list | query <词> | add <内容> | edit <fp> <新内容> | remove <fp> | undo <fp>
/memory pin <fp> | unpin <fp> | entries [词]
/memory compact [--dry-run] | dump
```

---

## 数据与隐私

- **全部本地**：零网络请求、零外部 API、零遥测
- **数据位置**：`<DSH_BIOMEMORY_DIR || ~/.dsh/biomemory>/biomemory.db`
- **删除机制**：墓碑式删除，可恢复
- **环境变量**：
  - `DSH_BIOMEMORY_DIR` - 数据目录
  - `DSH_MEMORY_ROOT` - Markdown 备份/日志目录

---

## 配置

配置文件：`~/.dsh/biomemory/biomemory.config.json`

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `nearDuplicateThreshold` | 0.7 | 写入去重的相似度阈值 |
| `nearDuplicateAction` | merge | 命中近重复时的处理方式 |
| `approvalFallback` | deny | 审批不可用时的策略 |
| `decayThreshold` | 3 | 低权重待处理阈值 |
| `hotTokenLimit` | 5000 | 冻结快照 token 上限 |
| `maxQueryResults` | 20 | 查询返回上限 |

---

## 设计原则

- **不做向量检索**：相同输入必须给相同结果，检索可解释
- **不做自动反思**：工作流只从重复中自然浮现
- **删除可回滚**：一切删除留墓碑
- **不做跨机同步**：保证文件可整体迁移

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v0.10.0 | 2026-10-05 | 重构为「记忆层」：触发式召回、类型优先级排序、五条清理规则、工作流升级、注入预算 |
| v0.9.2 | 2026-09-29 | 删死模块与死配置键 |
| v0.9.0 | 2026-09-29 | 移除嵌入模型，改单一确定性检索 |
| v0.8.0 | 2026-09-16 | 写入去重（bigram 相似度） |
| v0.6.0 | 2026-08-19 | 架构拆分 |

---

## License

MIT — 详见 [LICENSE](LICENSE)。