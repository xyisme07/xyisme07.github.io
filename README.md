# James 的投资笔记（静态博客 · 部署手册 / Runbook）

基于 **Quartz** 的静态博客：在 **Obsidian** 写作 → 推送到 **GitHub** → **GitHub Pages** 自动构建并上线。

- **免费**：Quartz 开源、GitHub Pages 免费托管
- **稳定**：纯静态站点，无后端、无数据库
- **简单**：日常只在 Obsidian 写 → `git push` → 自动发布

---

## ⚠️ 必须先读：关于账号与最终地址

部署时使用的 GitHub Personal Access Token **实际属于账号 `xyisme07`**（不是 `jamesxu`）。

GitHub 的用户站点规则：仓库必须命名为 `<登录名>.github.io` 且由该账号拥有。因此：

| 项目 | 实际值 |
|------|--------|
| 部署用 GitHub 账号（token 归属） | **`xyisme07`** |
| 仓库 | **`xyisme07/xyisme07.github.io`**（public） |
| **线上地址** | **https://xyisme07.github.io/** |

> 也就是说，用当前 token 能跑通的干净用户站点是 `https://xyisme07.github.io/`。
> 若你坚持要地址是 `https://jamesxu.github.io/`，前提是你有一个**登录名为 `jamesxu`** 的 GitHub 账号，并改用那个账号的 token 重新部署（把本手册里所有的 `xyisme07.github.io` 换成 `jamesxu.github.io` 即可，其余不变）。

**遗留待清理项**：因 token 缺少删除权限，误建的一条空仓库 `xyisme07/jamesxu.github.io` 仍在（已关掉它的 Pages）。请在本机登录 `xyisme07` 后，到 GitHub 网页 **Settings → Repositories** 里手动删除它。

---

## 一、当前已交付状态（可逐项核对）

| 核查项 | 命令 / 方法 | 预期结果 |
|--------|------------|----------|
| 仓库存在且为 public | 浏览器打开 `https://github.com/xyisme07/xyisme07.github.io` | 能看到代码、README |
| 已推送 main 分支 | 仓库页面 → 分支下拉选 `main` | 可见 `quartz/`、`content/`、`quartz.config.ts`、`deploy.yml` |
| Pages 已启用 | 仓库 → **Settings → Pages** | Source = **GitHub Actions**，状态 `built` |
| Actions 部署成功 | 仓库 → **Actions** 标签页 | 最近一次 run 为绿色 ✓ `success` |
| 站点可访问（首页） | 浏览器打开 `https://xyisme07.github.io/` | 显示博客首页 |
| 站点可访问（文章） | 打开 `https://xyisme07.github.io/Blog/welcome` | 显示欢迎文章 |

> 以上各项在交付时已全部验证通过（HTTP 200，内容与本地构建一致）。

---

## 二、本文件夹内容结构

```
jamesxu-blog/                      # ← 你的博客项目根目录（已初始化 git、已推送）
├── content/
│   ├── index.md                   # 首页
│   └── Blog/                      # 只有这个文件夹会被发布
│       ├── welcome.md
│       └── quartz-features-demo.md
├── quartz/                       # Quartz v4.5.2 框架源码（官方仓库原样）
├── quartz.config.ts              # 站点配置（已定制：中文标题 / zh-CN / 关闭外部追踪 / 系统字体）
├── quartz.layout.ts              # 页面布局（已定制页脚）
├── package.json                  # 依赖与脚本（框架自带）
├── package-lock.json
├── tsconfig.json                 # 构建配置（框架自带）
├── .nvmrc                        # 指定 Node 22
├── .gitignore                    # 已排除 node_modules/、public/
├── .github/
│   └── workflows/
│       └── deploy.yml            # GitHub Pages 自动部署工作流（核心）
└── README.md                     # 本手册
```

> `node_modules/` 和 `public/`（构建产物）已通过 `.gitignore` 排除，不会进仓库。

---

## 三、部署架构与自动发布链路

```
你在 Obsidian 写 → 把 Blog/ 的 .md 放进 content/Blog/ → git push
        │
        ▼
GitHub 仓库 xyisme07/xyisme07.github.io（main 分支）
        │  push 触发
        ▼
GitHub Actions（deploy.yml）
   ① checkout（fetch-depth:0）        ② npm ci
   ③ npx quartz build  → public/      ④ 上传 Pages 产物
        │
        ▼
GitHub Pages 发布  →  https://xyisme07.github.io/
```

> 你只在最左边写；后面全自动。构建+上线通常 1–2 分钟。

---

## 四、一次性搭建（若要从零在本机重做）

> 前提：本机已装 **Git** 与 **Node 22+**。本交付物已在本机/编写环境完成搭建与推送，此节供你复核或重建。

**第 1 步：克隆官方 Quartz 仓库（它自带 `quartz/` 源码与默认配置）**

```bash
git clone https://github.com/jackyzha0/quartz.git jamesxu-blog
cd jamesxu-blog
```
> 验证：`ls quartz/` 能看到 `cfg.ts` 等文件；`cat package.json` 显示 `"name": "quartz"`。

**第 2 步：放入定制文件（本交付物已含，重做时直接复制）**

- 覆盖 `quartz.config.ts`、`quartz.layout.ts`（已在项目中）
- 用本项目的 `content/` 替换官方 `content/`（只保留要发布的 `Blog/` 与 `index.md`）

**第 3 步：安装依赖 + 本地验证构建**

```bash
npm ci
npx quartz build
```
> 验证：输出 `Emitted NN files to public`，并在 `public/` 生成 `index.html`。
> 若报插件错误，先跑 `npx quartz plugin install --latest` 再构建。

**第 4 步：推送到你的 `用户名.github.io` 仓库**

```bash
git branch -M main
git remote add origin https://github.com/xyisme07/xyisme07.github.io.git
git add -A
git commit -m "init blog"
git push -u origin main
```
> 推送时若提示认证：用 GitHub 账号用户名 + 你的 PAT（`ghp_...`）作为密码；或在远程 URL 里嵌入 token（见第六节安全说明，**用后务必清理**）。

**第 5 步：开启 GitHub Pages（部署方式 = GitHub Actions）**

1. 仓库 → **Settings → Pages**
2. **Build and deployment → Source** 选 **GitHub Actions**
3. 保存。回到 **Actions** 标签页，确认 `deploy.yml` 自动运行并变绿。
> 也可以让一次 `git push` 自动触发后，Pages 会自动检测到 Actions 部署。若未自动启用，手动按上面步骤开启即可。

---

## 五、日常写作与发布（你的常态操作）

1. 在 Obsidian 的 `Blog/` 文件夹写笔记。
2. 把 `Blog/` 下的 `.md` 放进本项目的 `content/Blog/`。
   - **方式 A（最简）**：把 `content/Blog/` 当作发布副本，写完手动复制过去。
   - **方式 B（自动）**：用软链接（`mklink` / `ln -s`）把 Obsidian 的 `Blog/` 指向本项目的 `content/Blog/`。
3. 提交并推送：
   ```bash
   git add -A
   git commit -m "新增文章：xxx"
   git push
   ```
4. 等 1–2 分钟，打开 `https://xyisme07.github.io/` 即可看到更新。

**本地预览（不上线也能看效果）：**
```bash
npx quartz build --serve     # 默认 http://localhost:8080
```

---

## 六、凭证与安全（务必遵守）

GitHub **自 2021 年起不再接受账号密码推代码**，必须用 **Personal Access Token (PAT)**。

**生成 PAT（classic）：**
1. github.com → 头像 → **Settings** → 左下 **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. **Generate new token (classic)** → Note 填 `jamesxu-blog` → Expiration 选 **7 days**（短一点更安全）
3. 勾选 ☑ `repo`（全选）+ ☑ `workflow`
4. **Generate token** → 复制那串 `ghp_...`

**安全使用原则：**
- 本手册与项目文件里**绝不保存真实 token**，一律用占位符 `YOUR_GITHUB_PAT`。
- 推送时若想把 token 写进远程 URL（`https://YOUR_GITHUB_PAT@github.com/...`），**推送完成后立即改回**：
  ```bash
  git remote set-url origin https://github.com/xyisme07/xyisme07.github.io.git
  ```
- 用完即废：在 **Developer settings → Tokens (classic)** 里把该 token **Revoke**。

> 交付时使用的 token 已在部署完成后失效范围最小化；建议你**现在就去 Revoke 那条 `ghp_...`**，并如上重新生成一条自用。

---

## 七、选择性发布与隐私

本博客**只发布 `content/Blog/`**。Obsidian 其他私人笔记不进本项目，不会外泄。

- 不想发布的草稿：在笔记 frontmatter 加 `draft: true`（构建时自动排除）。
- `quartz.config.ts` 的 `ignorePatterns` 已忽略 `private / templates / .obsidian / drafts`。

---

## 八、故障排查

| 现象 | 可能原因 | 处理 |
|------|----------|------|
| Actions 跑红（红叉） | 依赖装不上 / 构建报错 | 点开 run 看日志；常见是 Node 版本不对 → 确认 `.nvmrc` 为 22，或本地 `node -v` |
| 推送被拒 `Authentication failed` | 用了登录密码而非 PAT | 改用 PAT；或用 token 嵌 URL（用完清理，见第六节） |
| 推送超时 / 卡死 | 国内网络不稳 | 重试；或换网络；小仓库通常能推 |
| 站点 404 | Pages 未启用 / 分支名不对 | 确认 Settings→Pages Source=GitHub Actions，且分支是 `main` |
| 文章没更新 | 没 push / Actions 没触发 | `git log` 确认已 push；Actions 里有新 run |
| `couldn't find git repository for content` | 本地没 git 历史 | 不影响线上（线上有 git 历史）；本地想消除警告就 `git init` |

---

## 九、国内访问加速（分两阶段）

- **阶段一（当前）**：GitHub Pages 直托，免费、部署顺。国内多数能开，偶发慢/打不开——这是 `github.io` 子域名免费方案的固有局限（你控制不了它的 DNS，套不了 CDN）。
- **阶段二（如需根治）**：买自定义域名（几十元/年）→ 在域名解析处指向 GitHub Pages（或前面套**国内 CDN 回源**，如又拍云/七牛/腾讯云 CDN）→ 做 **ICP 备案**（免费，约 1–2 周）。域名 + 备案才是真正的"国内加速"开关；前面项目完全复用，无需返工。

---

## 十、常用命令速查

| 目的 | 命令 |
|------|------|
| 本地构建 | `npx quartz build` |
| 本地预览 | `npx quartz build --serve`（http://localhost:8080） |
| 发布 | `git push` |
| 拉取更新 | `git pull` |
| 看部署状态 | 仓库 → Actions 标签页 |
| 看站点开关 | 仓库 → Settings → Pages |
