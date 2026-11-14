# isana's glog — Quartz v5 个人博客

基于 [Quartz v5](https://quartz.jzhao.xyz/) 构建的个人数字花园 / 博客，通过 GitHub Actions 自动构建并发布到 GitHub Pages。

> “[One] who works with the door open gets all kinds of interruptions, but [they] also occasionally gets clues as to what the world is and what might be important.” — Richard Hamming

## 工程概览

| 项目     | 说明                                                                               |
| -------- | ---------------------------------------------------------------------------------- |
| 站点地址 | https://isanahh.github.io/                                                         |
| 仓库     | `git@github-isanahh:isanahh/isanahh.github.io.git`（远程名 `github-pages`）        |
| 主分支   | `main`（推送即触发部署）                                                           |
| 站点标题 | `isana's glog`（`quartz.config.yaml` → `configuration.pageTitle`）                 |
| baseUrl  | `isanahh.github.io`（用户主页仓库，故无子路径）                                    |
| 内容目录 | `content/`（Markdown 源文件，唯一需要日常编辑的地方）                              |
| 构建产物 | `public/`（由构建生成，已在 `.gitignore` 中忽略，不入库）                          |
| 配置     | `quartz.config.yaml`（站点与插件配置）、`quartz.config.default.yaml`（默认值参考） |
| 主题     | `github-light` / `github-dark`，字体走 Google Fonts + CDN 缓存                     |

### 目录结构

```
content/            博客内容（Markdown），如 index.md、WorkLogger/、lifeusage/
quartz/             框架源码（构建器、CLI、组件、插件装载器）
quartz.config.yaml  站点与插件配置
quartz.ts           构建入口（加载配置 → buildQuartz）
.github/workflows/  CI：deploy.yaml（构建 + 发布到 GitHub Pages）
public/             构建输出（忽略提交）
```

### 自动化部署流程

`.github/workflows/deploy.yaml`：

- 触发条件：`push` 到 `main`，或手动 `workflow_dispatch`
- 构建：Node 24 → `npm ci` → `npx quartz plugin install` → `npx quartz build`
- 发布：`actions/upload-pages-artifact` → `actions/deploy-pages`

插件全部来自 npm（`@quartz-community/*`、`@quartz-themes/core`），由 `package-lock.json` 锁定版本，因此不会生成 `quartz.lock.json`，`plugin install` 在 CI 中为无操作。

## 环境要求

- Node.js >= 22（`.node-version` 为 `v22.16.0`）
- npm >= 10.9.2（`.npmrc` 中启用了 `engine-strict=true`）

## 常用命令

### 安装依赖

```bash
npm ci                 # 按 package-lock.json 精确安装（CI 同款）
npm install            # 开发时安装/更新依赖
```

### 本地开发与构建

```bash
npx quartz build                       # 一次性构建到 public/
npx quartz build --serve               # 构建并启动本地预览（默认 http://localhost:8080）
npx quartz build --serve --watch       # 预览 + 监听改动自动重建（推荐写作时使用）
npx quartz build -d docs               # 构建 docs 目录（等价于 npm run docs）
npx quartz build -o dist               # 指定输出目录
npx quartz build -v --bundleInfo       # 详细日志 / 构建体积分析
```

### 新建与维护内容

```bash
npx quartz create      # 初始化/选择内容目录
npx quartz upgrade     # 升级 Quartz 框架（别名 update）
npx quartz restore     # 从缓存恢复 content 目录
npx quartz sync        # 与 GitHub 仓库同步内容
npx quartz info        # 打印版本、环境与插件信息（提 issue 时用）
```

直接编辑 `content/` 下的 Markdown 即可，例如 `content/index.md` 为首页。

### 插件管理

```bash
npx quartz plugin list                 # 列出已安装插件及启用状态
npx quartz plugin install              # 按 lockfile 或配置安装插件
npx quartz plugin install --from-config    # 强制按 quartz.config.yaml 安装
npx quartz plugin add github:owner/repo    # 添加 Git 来源插件
npx quartz plugin remove <name>            # 移除插件
npx quartz plugin enable <name>            # 在配置中启用插件
npx quartz plugin disable <name>           # 在配置中禁用插件
npx quartz plugin config <name>            # 查看/设置某插件选项
npx quartz plugin doctor                   # 校验配置、lockfile 与安装状态
npx quartz plugin prune                    # 移除配置中已不再引用的插件
npx quartz tui                             # 交互式插件管理器（需先 npx quartz plugin add github:quartz-community/tui）
```

### 代码质量

```bash
npm run check          # TypeScript 类型检查 + Prettier 格式校验
npm run format         # Prettier 自动格式化
npm test               # 运行 tsx 测试
```

### 提交与发布

```bash
git status                                  # 查看改动
git add content/                            # 暂存内容改动（或 . 全部）
git commit -m "docs: 新增 xxx 笔记"
git push github-pages main                  # 推送即触发 GitHub Actions 构建发布

# 首次关联远程（如尚未配置）
git remote add github-pages git@github-isanahh:isanahh/isanahh.github.io.git
```

> 使用 SSH 别名 `github-isanahh`，需在 `~/.ssh/config` 中配置对应用户的 Host 与密钥。

### 验证部署

```bash
# Actions 运行记录
https://github.com/isanahh/isanahh.github.io/actions

# 线上站点
https://isanahh.github.io/
```

若部署失败提示 `Get Pages site failed`，检查仓库 **Settings → Pages → Source** 是否已设为 **GitHub Actions**。

## 常见问题

- **改了内容但站点没更新**：确认推送到的是 `main` 分支（workflow 只监听 `main`），并查看 Actions 运行结果。
- **本地预览不刷新**：改用 `npx quartz build --serve --watch`。
- **依赖安装报引擎错误**：`.npmrc` 启用了 `engine-strict`，请升级 Node 至 >= 22。

## 许可与致谢

框架本体遵循 [MIT License](./LICENSE.txt)，版权归 [jackyzha0](https://github.com/jackyzha0) 及贡献者所有。
Quartz 文档：https://quartz.jzhao.xyz/ ｜ [Discord 社区](https://discord.gg/cRFFHYye7t)
