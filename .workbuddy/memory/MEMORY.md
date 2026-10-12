# M-Firefly 项目长期记忆

## 仓库与身份
- 仓库：`https://github.com/daylight5564/M-Firefly.git`（public），默认分支 `master`
- git 身份：`MoonLightWS <qq1755711298@163.com>`
- 线上域名：`https://wangshibo.dpdns.org`（见 `src/config/siteConfig.ts` 的 `site_url`）
- 项目：Astro + Firefly 主题博客；dev server 端口 **4321**

## ⚠️ 在本机推送代码（每次都要用，直接抄）
本机 `github.com` 有两道坎，必须同时绕过：
```bash
cd "/c/Users/a1522/Desktop/open/00M-blog/M-Firefly"
GCM="D:/Work-Buddy/WorkBuddy/resources/vendor/PortableGit/mingw64/bin/git-credential-manager.exe"
GIT_TERMINAL_PROMPT=0 git \
  -c http.sslVerify=false \
  -c http.schannelCheckRevoke=false \
  -c credential.helper= \
  -c credential.helper="$GCM" \
  push origin master
```
1. **证书吊销检查失败**（`CRYPT_E_NO_REVOCATION_CHECK 0x80092012`）：
   只加 `http.schannelCheckRevoke=false` 无效，必须同时 `http.sslVerify=false`。
   （`codeload.github.com` / `baidu.com` / `api.bilibili.com` 直连正常，仅 `github.com` 受影响；
   `curl --ssl-no-revoke` 也可正常访问）
2. **默认 `credential.helper=helper-selector` 在非交互环境取不到凭据**
   （报 `could not read Username for 'https://github.com'`）。
   Windows 凭据管理器里已存有 `git:https://github.com`（用户 `daylight5564`），
   显式指定 GCM 即可取出。注意用 `-c credential.helper=`（空值）先清空原列表，否则会叠加。

## 部署实况（与早期笔记不符，以此为准）
- 远端 **未启用 GitHub Pages**（`/repos/.../pages` 返回 404，`has_pages:false`），
  且 `actions/runs` 计数为 **0**（Actions 没跑过）。
  所以 `.github/workflows/deploy.yml`（GitHub Actions → Pages）**不是实际部署路径**。
- 仓库同时有 `vercel.json` 与 `wrangler.jsonc` / `wrangler.toml`
  （worker 名 `firefly`，assets 目录 `./dist`）。
  → 线上由 **Vercel 或 Cloudflare 自身的 GitHub 集成**在 push 后自动构建。
- 因此 push 到 `master` 后，需到对应平台查看构建结果。

## 关键约定与坑
- **配置默认值 vs 访客偏好**：所有显示设置都是
  `localStorage 有记录则优先，否则回退配置默认值`（`src/utils/setting-utils.ts` 的 `getStored*()`）。
  改了配置文件却在浏览器里看不出变化时，先清 localStorage 或用无痕窗口验证。
- **`displaySettingsConfig.ts` 只管面板项显隐**（`*Switchable`），不设默认值。
- **文章里的图片路径指向不存在的文件会让整个 dev server 全站 500**（报 `ImageNotFound`）。
  本地预览突然全站打不开时，先排查最近改过的文章里的图片路径。
- `src/content/posts/` 用 `**/*.{md,mdx}` 递归收集，**支持子目录**；
  文章 URL 由相对文件路径决定，schema 里**没有** `slug` 字段（写了会被静默丢弃）。
- frontmatter 必须用 ASCII 冒号 `:`，全角 `：` 会让整篇解析失败被丢弃。
- frontmatter 中 `updated:` / `seriesOrder:` **只写键不写值会变成 null 导致校验失败**。
- `.workbuddy/memory/` 已被纳入 git 跟踪（`2026-10-11.md` 在仓库里）。
- 工作区出现过被整体回滚的情况（本地未提交改动消失），重要改动要及时提交。
