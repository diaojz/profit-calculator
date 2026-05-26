# 维护说明

## 项目结构

```
profit-calculator/
├── index.html       # 全部业务代码（HTML + CSS + JS 都内联在这里）
├── CNAME            # GitHub Pages 域名标识（保留作为备份方案）
├── README.md
├── CONTRIBUTING.md
└── .gitignore
```

整个项目就一个 `index.html`，所有 CSS 和 JS 都内联其中。唯一的外部依赖是通过 CDN 加载的 `html2canvas`（用于导出 PNG）。

## 本地开发

直接编辑 `index.html`，浏览器刷新就能看到变化。

```bash
open -a "Google Chrome" index.html
```

## 发布更新（重要）

托管在 Vercel，绑定到 `https://coding.chengbei.org/`。

### 标准流程

```bash
cd "/Users/diaoye/Documents/BD/App Store/app/profit-calculator"

# 1. 提交代码到 GitHub（保留版本历史，可选）
git add -A
git commit -m "fix: 简短描述本次修改"
git push

# 2. 部署到生产环境（这一步是必须的）
vercel deploy --prod --yes
```

**关键提醒**：目前没接 git push 自动部署，必须手动跑 `vercel deploy --prod --yes` 才会上线。如果只 push 不 deploy，线上还是老版本。

### 想接 git push 自动部署？

到 Vercel 控制台 → `profit-calculator` 项目 → Settings → Git → Connect Git Repository → 选 `diaojz/profit-calculator`。接好之后 `git push` 就会自动触发 Vercel 部署，不再需要本地跑 `vercel deploy`。

## 域名与 DNS

| 项目 | 配置 |
|---|---|
| 域名 | `coding.chengbei.org` |
| 注册商 / DNS | Cloudflare |
| 解析记录 | CNAME `coding` → `cname.vercel-dns.com`（DNS-only / 灰云）|
| 证书 | Vercel 自动签发 Let's Encrypt，无需手动管理 |

DNS 改动可在 Cloudflare 后台手工操作，或者用 `app/chengbei-fleet/.env` 里的 `CF_TOKEN` 走 API。

## 业务规则（写在代码里）

这些是固定常量，要调整就直接改 `index.html` 里的 `compute()` 函数：

- **销售提成**：`总收入 × 10%`
- **团队分账**（占剩余可分）：
  - 主讲老师 40%
  - 技术支持 30%
  - 课程负责 20%
  - 课程管理 10%

## 数据存储

数据完全存在浏览器 `localStorage`，没有服务器侧持久化。两个 key：

| Key | 内容 |
|---|---|
| `profit-calc-state-v1` | 当前输入的草稿（每次输入后自动保存） |
| `profit-calc-history-v1` | 已归档的历史记录数组 |

清浏览器缓存或换电脑会丢，所以历史记录功能本质上是"本机草稿盒"，不是云同步。

## 外部依赖

唯一的外部资源：`html2canvas` 1.4.1（cdnjs CDN，用于导出 PNG）。

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
```

CDN 挂了的话，"导出图片"功能会提示失败，其他功能（计算、复制、历史记录）不受影响。

## 视觉规范

暖米编辑风（cream bento 风格）。颜色和字体都用 CSS 变量统一管理，集中在 `:root`：

| 变量 | 用途 | 当前值 |
|---|---|---|
| `--bg` | 页面底色 | `#ece7de` |
| `--card` | 卡片背景 | `#fcfaf5` |
| `--ink` | 主文字 | `#2d2620` |
| `--accent` | 强调线 / 焦点框 | `#c8956d` |
| `--accent-soft` | 米黄高亮（总收入框 + 剩余可分） | `#ffe69e` |

字体用系统字（`-apple-system / PingFang SC`），数字一律 `font-variant-numeric: tabular-nums` 等宽对齐。

要换色调就改 `:root` 里的变量，不要散落到具体规则里写死颜色。
