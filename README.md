# WeChatCodex

在微信里直接使用 Codex。

`WeChat -> ilink / ClawBot 协议 -> 本地 Node.js daemon -> codex exec / codex exec resume`

这个项目把个人微信接到本机 `codex` CLI，不走 OpenClaw Gateway，不需要公网 IP，也不需要域名。整体方法受到文章《在微信里使用 Claude Code，刚刚在 GitHub 上开源了这个 Skill。》启发，但执行层改成了 Codex CLI，并针对 Codex 的运行模型做了适配。

> 作者：[AI大同学](https://aifellow.ai) ｜ 上手记录：[我把 Codex 接进了微信](https://aifellow.ai/articles/tc01)

## 最新更新

- 已支持微信图片、语音、音频文件、视频消息
- 语音消息优先使用微信侧转写文本
- 音频文件会自动下载、转写后再交给 Codex
- 视频会自动提取语音转写，并抽取关键帧图片一起交给 Codex
- 当上一条消息还在处理中时，新消息不会再被直接丢掉，而是进入队列顺序处理
- 音频转写超时已改为按音频时长动态放宽，3 分钟以上音频不再因为固定 120 秒门限被提前中断

## 功能

- 微信文字消息直接发给本机 Codex，并通过 `thread_id` 保持多轮上下文
- 支持图片消息，自动落地后转给 Codex `--image`
- 支持微信语音消息，优先使用微信侧转写文本
- 支持音频文件和视频消息，自动做本地转写；视频会额外抽取关键帧给 Codex
- 支持 slash 命令管理模型、工作目录、运行模式和会话状态
- 守护进程常驻运行，macOS 使用 `launchd`，Linux 提供 `systemd` / `nohup` 路径
- 本地持久化账号、配置、日志和会话
- 项目过程文档按 Superpowers 工作流保存在 `docs/superpowers/`

## 为什么不是 Gateway 方案

- 直接复用微信 ilink / ClawBot 协议层
- 不接 OpenClaw Gateway
- 消息在本机常驻进程内完成接收、转发和回包
- Codex 侧采用 `codex exec` / `codex exec resume`

和参考文章里的 Claude Code Skill 不同，这里没有逐工具 `y/n` 授权回调，因此运行权限采用显式模式切换。

## 运行模式

- `plan`: 只读分析模式
- `workspace`: 工作区可写模式
- `danger`: 无沙箱模式

默认模式是 `workspace`。

## 前置条件

- Node.js 18+
- macOS 或 Linux
- 本机已安装并完成登录的 `codex`
- 如需处理音频或视频，需额外安装 `ffmpeg`、`ffprobe` 和 `whisper`
- 已开通并可使用的个人微信账号

## 安装

```bash
git clone https://github.com/DrDavidDa/WeChatCodex.git
cd WeChatCodex
npm install
```

## 首次设置

```bash
npm run setup
```

执行后会弹出二维码。扫码绑定微信后，输入默认工作目录即可。

## 启动与管理

```bash
npm run daemon -- start
npm run daemon -- status
npm run daemon -- restart
npm run daemon -- stop
npm run daemon -- logs
```

## 微信命令

| 命令 | 作用 |
| --- | --- |
| `/help` | 查看帮助 |
| `/clear` | 清空当前会话 |
| `/status` | 查看当前会话状态 |
| `/model <name>` | 切换 Codex 模型 |
| `/cwd <path>` | 切换工作目录 |
| `/mode <plan|workspace|danger>` | 切换执行模式 |
| `/skills` | 查看可用技能 |
| `/<skill> [args]` | 触发某个技能 |

## 数据目录

默认数据目录为 `~/.wechat-codex-bridge/`。为了兼容当前已运行的实例，目录名暂时保留旧的 bridge 命名。

```text
~/.wechat-codex-bridge/
├── accounts/
├── sessions/
├── logs/
├── tmp/
├── get_updates_buf
└── config.env
```

## 当前状态

- 主要在 macOS 上完成了实际扫码、收发消息和守护进程验证
- Linux 路径已提供脚本支持，欢迎补充验证反馈
- 已修复守护进程被长时间 `codex exec resume` 阻塞后无法继续轮询的问题
- 音频自动转写默认只处理 10 分钟内的音频/视频，超出会明确提示截短后重发

## 灵感与致谢

- 这次项目受到文章《在微信里使用 Claude Code，刚刚在 GitHub 上开源了这个 Skill。》启发
- 方法论上参考了 `Wechat-ggGitHub/wechat-claude-code` 对微信协议层与本地守护进程的拆法
- 项目实现已经替换为面向 Codex CLI 的桥接方案

## 开发

```bash
npm run build
npm test
```

## License

[MIT](LICENSE)
