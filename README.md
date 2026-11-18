# dsh-godot-preset

给 DeepSeek Harness（DSH）用的 **agent preset bundle**：在 `standard` 预设的全部能力之上，追加
Godot Breakpoint MCP 客户端，提供编辑器实时状态、运行中游戏调试、GDScript / C# 的 LSP 与断点调试工具。

## 内容

| 文件 | 作用 |
|---|---|
| `package.json` | bundle 清单；`dsh.bundle.patch` 指向 `cordis.patch.yml` |
| `cordis.patch.yml` | 一条 `- insert:` 声明 Loader 行 `preset-godot`（`@deepseek-ai/dsh-agent-preset`） |

预设声明（`config`）：

- `id: godot`，Loader 行 id 为 `preset-godot`
- `name: Godot 开发（含 MCP）`，`order: 2`
- `plugins` = `@deepseek-ai/dsh-web-app` 的 `presets/standard.patch.yml` 插件列表，**末尾追加**：

```yaml
- id: mcp-breakpoint
  name: '@deepseek-ai/dsh-mcp-client'
  config:
    serverName: godot
    transport: stdio
    command: D:/DevTools/runtime/nodejs/node.exe
    args: [D:/DevTools/runtime/nodejs/node_modules/breakpoint-mcp/dist/index.js]
    env: { GODOT_PROJECT: 'D:/project/project-d', ... }
```

预设行另带 `inject: [biliBasicCompactionPolicy]`，与 `billion-context` 对
`preset-standard` / `preset-ptc` / `preset-minimal` / `preset-cordis` 的注入保持一致。
**因此本 bundle 依赖 profile 内装有 `billion-context`**：若把它移除，本预设行会停在 pending 不激活。

## 安装

DSH 的预设只能由 bundle 的 patch 声明或覆盖，UI 里没有直接新增预设的入口。

```text
Plugins > Add plugin，target 填本仓库目录的绝对路径
```

或让 agent 调 `plugin_manager`，`action: install_bundle`，`target` 填**绝对目录**。

从远程安装：`install_bundle` 只接受本地绝对目录，所以先克隆再装。

```bash
git clone https://github.com/dtsummery/dsh-godot-preset.git D:/DSH_workspace_flash/godot-preset
# 然后在 DSH 里对该目录执行 install_bundle
```

## 注意

`cordis.patch.yml` 的插件列表是 `standard` 的副本。DSH 升级后 `standard` 若增删插件，
本文件不会自动跟随，需要手工同步（文件头注释亦已标注）。
