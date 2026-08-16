# dsh-restart-plugin

DSH 网页插件：在 DSH 的 **设置 → 通用设置** 中添加两行一键操作——「关闭后台服务」与「刷新前端」。已发布至 [dsh-plugin](https://github.com/topics/dsh-plugin) 主题。

> A DeepSeek Harness web plugin: adds **"Shut down backend service"** and **"Refresh frontend"** rows to Settings → General, with topmost confirm/cancel dialogs (confirm = black button, cancel = white button).

## 功能

| 操作 | 行为 |
| --- | --- |
| **关闭后台服务** | 弹出置顶确认框（确认=黑底白字，取消=白底黑字）；确认后后台**优雅退出**，并在对话框里显示手动重启命令。之后由你在终端重新运行该命令、重新打开页面即可。热插件会随关闭消失（对话框已提示）。 |
| **刷新前端** | 重新加载页面，且**保留创造模式的热插件**（重新挂载所有活跃动态插件的客户端半边，Host 半边原样不动）。后台不重启。**直接按 F5 或 Ctrl+R 效果相同**——由页面代码强制刷新（不依赖浏览器默认行为），页面卡死时无需打开设置即可刷新并保留热插件。 |

## 安装

把本仓库集成到你的 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 源码工作区（本插件按源码方式安装）：

1. 将 `packages/host/restart` 与 `packages/client/ui-settings-restart` 两个目录复制到工作区的 `packages/host/` 与 `packages/client/` 下；
2. 在工作区根目录执行 `git apply install.patch`（含 web-app 组合注册、api-remotes 挂载、ui-cordis 热插件重挂载与 tsconfig 引用；如版本有出入请对照补丁手工合并）；
3. 安装与构建：

```powershell
pnpm install
pnpm exec tsc -b tsconfig.host.json
pnpm exec tsc -b tsconfig.client.json
pnpm --filter @deepseek-ai/dsh-host-restart bundle
pnpm --filter @deepseek-ai/dsh-client-ui-settings-restart bundle
pnpm --filter @deepseek-ai/dsh-api-remotes bundle
pnpm --filter @deepseek-ai/dsh-client-ui-cordis bundle
```

4. 手动重启后台（例如 `pnpm dsh web`），刷新页面后即可在 **设置 → 通用设置** 最下方看到两行按钮。

> 两个包的 `lib/` 产物（含 typert 契约）已随仓库附带，通常无需重新生成。

## 工作原理

- **关闭后台**：Host 端 `restart` Remote 调用启动器的 `appExit(0)` 优雅退出通道——不创建任何进程、无脚本解析，路径上没有任何可失败环节。
- **刷新前端**：确认时写入一次性 `sessionStorage` 标记；页面重载后由 `ui-cordis` 消费该标记，对每个活跃且有客户端半边的动态插件调用已有的"附加"运行路径，仅恢复界面、不重启 Host 半边。
- 对话框通过 `createPortal` 渲染到 `document.body`（z-index 2000），始终位于最顶层。

## 要求

- DeepSeek Harness 源码工作区（本插件按 `0.1.0-rc.5` 版结构集成）
- Node.js 22+、pnpm

## 相关项目（双向互链）

- [dsh-plugin-suite](https://github.com/zdjmrq/dsh-plugin-suite) — 定制插件套件（局部 fork），**内置本插件**与 `dsh-careful-full-access` 命令守卫（防误删）的完整累计补丁 `install.patch`。想一次性获得两个插件的全部功能：克隆上游 → `git checkout 47f943859b` → `git apply install.patch` → `pnpm install` → `pnpm run build` → `pnpm dsh web`。本仓库的 `install.patch` 仅用于把「关闭/刷新前端」功能单独装进其它 harness 工作区；
- [dsh-text-open-source](https://github.com/zdjmrq/dsh-text-open-source) — 「文字开源」枢纽仓库:本插件的可复刻文字描述见 [plugins/dsh-restart-plugin.md](https://github.com/zdjmrq/dsh-text-open-source/blob/main/plugins/dsh-restart-plugin.md)(不依赖代码即可复刻、便于理解与微调);
- [deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) — 官方上游。

## License

MIT
