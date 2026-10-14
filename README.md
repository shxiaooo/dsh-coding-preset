# dsh-coding-preset

DSH 自定义 Agent 预设：**编码模式**。

安装后出现在 DSH 预设列表的 **Custom** 分类下，与官方预设并列可选。

## 这是什么

DSH 内置的 `standard` 预设包含完整能力，其中有些对纯编码任务并不需要。
本预设就是 `standard` **减去**三类东西：

| 去掉的能力 | 说明 |
|---|---|
| 目标（goal）命令与工具 | 编码任务不需要设定长期目标 |
| 工作流（workflow）引擎与工具 | 不跑多步编排流程 |
| 子 Agent 控制工具 | 不开放子 Agent 的创建与管理 |

保留的是编码任务真正用得到的：**文件读写、搜索、shell、Skills、计划模式、提醒、上下文压缩**。

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

- **子 Agent 保留为一次性委派**（`subagent` / `subagent_fork`），但过滤掉了 schedule 系列工具，
  子 Agent 无法自行创建定时任务。
- **Shell 工具按平台切换**：Windows 下启用 `tool-pwsh`、关闭 `tool-bash`；其他平台相反。
- **插件管理器工具默认关闭**。
- 预设的提示词由各插件自带，本文件只做能力声明，不写提示词正文 ——
  因此不会出现「描述了某个本预设没有的能力」这种情况。
