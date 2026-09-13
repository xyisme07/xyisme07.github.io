# James 的投资笔记（静态博客）

基于 **Quartz** 的静态博客：在 **Obsidian** 写作 → 推送到 **GitHub** → **Cloudflare Pages** 自动构建并上线。

- **免费**：Quartz 开源、Cloudflare Pages 免费额度充足
- **稳定**：纯静态站点，无后端、无数据库
- **简单**：日常只在 Obsidian 写 → `git push` → 自动发布

---

## 一、本文件夹里已经为你准备好的内容

```
jamesxu-blog/
├── content/
│   ├── index.md                 # 首页
│   └── Blog/                   # 只有这个文件夹会发布
│       ├── welcome.md
│       └── quartz-features-demo.md
├── quartz.config.ts            # 站点配置（已按你定制：中文标题/zh-CN/关闭外部追踪/系统字体）
├── quartz.layout.ts            # 页面布局（已定制页脚）
├── tsconfig.json
├── .nvmrc                      # 指定 Node 22
├── .gitignore
└── README.md
```

> 说明：Quartz 框架源码 `quartz/`（来自官方仓库）这一步需要在你**本机正常终端**拉取（当前编写环境无法访问 GitHub）。下面第二步就是这件事，几分钟搞定。

---

## 二、一次性搭建（在本机终端执行）

> 前提：已装 Node 22+ 与 Git，且有 GitHub / Cloudflare 账号。

```bash
# 1) 克隆官方 Quartz 仓库（它自带 quartz/ 框架源码与默认配置）
git clone https://github.com/jackyzha0/quartz.git jamesxu-blog
cd jamesxu-blog

# 2) 用本文件夹里的定制文件覆盖（直接复制粘贴即可）
#    覆盖：quartz.config.ts、quartz.layout.ts
#    替换：把官方 content/ 整个换成这里的 content/（含 Blog/）
#    （package.json / tsconfig.json 用克隆仓库自带的即可，不必替换）

# 3) 安装依赖并本地验证能否构建
npm install
npx quartz build          # 成功会在 public/ 生成站点

# 4) 推送到你的 GitHub 仓库 jamesxu.github.io
git remote set-url origin https://github.com/jamesxu/jamesxu.github.io.git
git add -A
git commit -m "init blog"
git branch -M main
git push -u origin main
```

> 若 `npx quartz build` 报插件相关错误，先执行一次 `npx quartz plugin install --latest` 再构建。

---

## 三、连接 Cloudflare Pages（一次性，之后全自动）

1. 登录 Cloudflare → **Workers & Pages** → **Create** → **Pages** → **连接到 Git** → 选 `jamesxu.github.io`。
2. 构建设置：
   - 框架预设：**None**
   - 构建命令：`npx quartz build`
   - 输出目录：`public`
   - 环境变量（可选，保险）：`NODE_VERSION = 22`
3. 保存并部署。以后每次 `git push` 自动重新构建上线（约 1–2 分钟）。

> 项目根目录的 `.nvmrc` 已写 `22`，Cloudflare 会自动读取，通常无需手动设环境变量。

---

## 四、日常写作流程（Obsidian）

1. 在 Obsidian 的 `Blog/` 文件夹里写笔记。
2. 把 `Blog/` 下的 `.md` 同步进本项目的 `content/Blog/`。
   - **方式 A（最省心）**：直接把 `content/Blog/` 当成发布副本，写完手动复制过去。
   - **方式 B（自动）**：用软链接（`mklink` / `ln -s`）把 Obsidian 的 `Blog/` 指向本项目的 `content/Blog/`，或写一行同步脚本。
3. 提交并推送：
   ```bash
   git add -A
   git commit -m "新增文章：xxx"
   git push
   ```
4. 等 1–2 分钟，网站自动更新。

---

## 五、选择性发布与隐私

本博客**只发布 `content/Blog/` 里的内容**。Obsidian 其他私人笔记不会进入本项目，因此不会外泄。

- 不想发布的草稿：在笔记 frontmatter 加 `draft: true`（构建时自动排除）。
- 整个私密文件夹也可排除：已在 `quartz.config.ts` 的 `ignorePatterns` 里忽略 `private / templates / .obsidian / drafts`。

---

## 六、国内访问加速（分两阶段）

- **阶段一（当前默认）**：Cloudflare Pages 直托，免费、部署顺滑；国内多数能开，偶发慢。
- **阶段二（如需根治）**：买自定义域名（几十元/年）→ 在 Cloudflare Pages 绑定自定义域名 → 前面套**国内 CDN 回源**（又拍云 / 七牛 / 腾讯云 CDN）→ 做 **ICP 备案**（免费，约 1–2 周）。这一步与主机选择无关，是真正的"国内加速"开关。前面搭的项目完全复用，无需返工。

> 提醒：免费 Cloudflare **没有中国节点**，所以"部署方便"≠"国内快"。阶段二是治本手段。

---

## 七、常用命令

| 目的 | 命令 |
|------|------|
| 本地构建 | `npx quartz build` |
| 本地预览 | `npx quartz build --serve`（默认 http://localhost:8080） |
| 发布 | `git push` |
| 拉取更新 | `git pull` |
