# dsh-coding-preset

DSH 自定义 Agent 预设：**编码模式**。

安装后出现在 DSH 预设列表的 **Custom** 分类下，与官方预设并列可选。

## 这是什么

DSH 内置的 `standard` 预设包含完整能力，其中有些对纯编码任务并不需要。
本预设就是 `standard` **减去**两类东西：

| 去掉的能力 | 说明 |
|---|---|
| 目标（goal）命令与工具 | 编码任务不需要设定长期目标 |
| 工作流（workflow）引擎与工具 | 不跑多步编排流程 |

保留的是编码任务真正用得到的：**文件读写、搜索、shell、Skills、计划模式、提醒、上下文压缩**。

## 与 standard 的差异

除上表两项外，还有两处刻意不同：

| 项目 | standard | 编码模式 |
|---|---|---|
| 子 Agent 控制工具 | `interrupt: false`，无 `interrupt_agent` | `interrupt: true`，四个工具齐全 |
| 插件管理器工具 | 有 profile 时启用 | 恒关闭 |

**子 Agent 控制工具是保留并加强的**：`send_message`（追加指示）、`interrupt_agent`
（打断当前轮次）、`list_agents`（查看子 Agent 树）、`wait_agent`（等待输入）全部可用。
`standard` 预设把 `interrupt` 设为 `false`，本预设改为 `true` —— 编码任务里派出去的子 Agent
最需要中途纠偏，而 `subagent` 工具的描述本身就提到 `send_message`，不装这些工具会让描述
指向一个不存在的工具。

## 安装

```sh
pnpm dsh plugin --profile web add link:~/dsh-coding-preset
```

装完**重启** DSH，预设列表的 Custom 分类下就会出现「编码模式」。

## 配置

预设定义在 [`cordis.patch.yml`](cordis.patch.yml)，通过 `package.json` 里的
`dsh.bundle.patch` 挂载。要调整预设内容直接编辑该文件。

修改后重启 DSH 生效。

## 说明

- **子 Agent 委派可用**（`subagent` / `subagent_fork`），并保留消息、打断、发现、等待四类控制工具。
  子 Agent 继承父 Agent 的预设，因此也拥有 schedule 系列工具。
- **Shell 工具按平台切换**：Windows 下启用 `tool-pwsh`、关闭 `tool-bash`；其他平台相反。
- **插件管理器工具默认关闭**。需要用 Agent 安装插件时请切回 `standard`。
- 预设的提示词由各插件自带，本文件只做能力声明，不写提示词正文。

## 本地修改记录（适配 DSH 0.2.2-rc.3）

本副本在上游 v1.0.0 基础上做了以下修改：

1. **`tool-subagent` 配置字段更新**（上游写法会激活失败）
   - 移除 `modelSelectionSettings: true`（该字段已不存在）
   - 移除两个工具实例上的 `toolFilter.deny`（新版会直接抛
     `tool-subagent: toolFilter is no longer supported`）
   - 改用当前版本的 `modelSelection: true` + `modelDiscovery: true`
2. **补回内置 `dsh-doc` skill 的目录**
   上游没有声明 `customSkillDirs`，而 `standard` 预设是靠这一行挂载内置 skill 的。
   本 bundle 位于安装目录之外，`baseUrl` 相对解析够不到安装目录的 `node_modules`，
   因此改为探测 `DSH_INSTALL_DIR` 与 `process.resourcesPath` 两个已知位置，
   只贡献实际存在的目录；位置变化时降级为「没有额外 skill 根」，而不是整个预设激活失败。
3. **补上 `tool-office`**
   与 `standard` 对齐，避免「编码模式」缺少 Office 工具。
4. **加回子 Agent 控制工具，并开启 `interrupt`**
   上游把 `tool-subagent-control` 整组删掉，导致子 Agent 只能一次性委派、无法中途干预，
   且 `subagent` 的描述仍会提到 `send_message`。现改为保留四个工具并把 `interrupt`
   设为 `true`（`standard` 是 `false`）。

`toolFilter` 移除后，子 Agent 不再被限制创建定时任务。新版把子 Agent 的能力交给
「子 Agent 使用哪个预设」来管理，`toolFilter` 已无对应机制。
