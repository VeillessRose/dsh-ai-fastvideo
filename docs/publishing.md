# 发布与分发（AI 速成视频 / dsh-ai-fastvideo）

本文只讲「怎么把它装到别人的 DSH 里」，分四种形态，从轻到重：

| 形态 | 适合谁 | 一条命令/一步操作 | 是否会出现在「插件市场」 |
|---|---|---|---|
| A. 本地目录 `install_bundle` | 自己开发、内网机器 | `plugin_manager install_bundle target=<目录>` | 否 |
| B. GitHub 仓库 + 本地 clone | 少量朋友、公司内共享 | `git clone` 后同 A | 否 |
| C. npm 包 | 公开分发（推荐） | `dsh plugin --profile web add dsh-ai-fastvideo` | 需再做 D 才会被市场发现 |
| D. 进社区市场目录 | 想让所有人 `Discover` 到 | 给 `awesome-dsh-plugin` 提 PR | 是 |

## 0. 命名约定（`dsh-plugins-` 是不是标准？）

不是。DSH 只要求「能作为 npm 包安装、带 `dsh.bundle.patch`」即可，包名由作者决定：

- 官方/社区当前的主流是 `dsh-xxx`（`dsh-context`、`dsh-image-gen`、`dshmarket`、`dsh-better-sidebar`）。
- 带 scope 的是 `@scope/dsh-xxx`（`@smalltailqwq/dsh-client-ui-skin-maid-atelier`）。
- `dsh-plugins-xxx` 只是某个作者的私有前缀（市场里 5126 条中只有 5 条），并非约定。

所以本包用 `dsh-ai-fastvideo` 是合规的，而且这个 npm 名字**当前未被占用**。

## 1. 发 npm 包（形态 C）

```powershell
cd F:\path\to\dsh-ai-fastvideo
npm login                      # 第一次
npm pack --dry-run             # 看会打包哪些文件（必须是 files[] 里列的）
npm publish --access public    # 公开包
```

发完别人就能：

```powershell
dsh plugin --profile web add dsh-ai-fastvideo
```

装完在 DSH 里会看到 profile 的 `node_modules/dsh-ai-fastvideo`（`package.json.source` 显示 `dsh-ai-fastvideo@0.2.0`），
与 `dsh plugin --profile web add dsh-context` 完全同款。

**版本号**：改 `package.json` 的 `version` 后重新 `npm publish`；用户跑 `dsh plugin --profile web add dsh-ai-fastvideo@latest` 或 `npm update` 升级。

## 2. 进社区市场（形态 D）

市场不是"发布即收录"，它读的是一个 curated 索引：

- npm 包 `dsh-plugin-catalog`（仓库 `https://github.com/awesome-dsh-plugin/awesome-dsh-plugin`）
- 索引 JSON：`https://cdn.jsdelivr.net/npm/dsh-plugin-catalog/index.json`（另有 `awesome-dsh-plugin.com` / `dshmarket.com` 两个镜像）
- 结构：`{ "format": 1, "plugins": [ { "name", "title", "description": { "zh", "en" }, "homepage", "repository", "metadata", "versions" } ] }`

收录步骤：

1. Fork `awesome-dsh-plugin`。
2. 在它的 `plugins.json`（打包进 `dsh-plugin-catalog`）里按现有条目格式加一条本包记录，`name` 必须与 npm 包名一致。
3. 提 PR，说明包做什么、依赖什么（本包要求本机有 ffmpeg + 一个 Chromium 内核浏览器）。
4. 合并后发新版 `dsh-plugin-catalog`，市场拉到新索引就能搜到。

> 想让兼容性徽标正常显示，`package.json` 里要带 `engines.dsh` 与 `dsh.compatibility.dshReleases`（本包已带）。

## 3. 用户装完之后要配什么

只有一件必做的事：填一个 OpenAI 兼容的模型 key（用于「按风格提示词写页面」和「AI 自拟分镜」）。

```jsonc
// Windows: %USERPROFILE%\.dsh\data\dsh-ai-fastvideo\config.json
// macOS/Linux: ~/.dsh/data/dsh-ai-fastvideo/config.json
{
  "llm": { "baseUrl": "https://api.deepseek.com/v1", "model": "deepseek-chat", "apiKey": "sk-..." }
}
```

其余（python / ffmpeg / 浏览器）都是自动探测，探测不到才需要写进 `config.json`：

```jsonc
{
  "python": "C:\\Python311\\python.exe",
  "ffmpeg": "C:\\ffmpeg\\bin\\ffmpeg.exe",
  "ffprobe": "C:\\ffmpeg\\bin\\ffprobe.exe",
  "browser": "C:\\Program Files (x86)\\Microsoft\\Edge\\Application\\msedge.exe",
  "styleDir": "D:\\my-styles",
  "voices": [{ "id": "zh-CN-XiaoxiaoNeural", "name": "晓晓（女）" }]
}
```

打开面板 → 「AI 速成视频」页签 → 右上「环境」角标可以看每一项探测结果（`/ai-video/health` 的 `pythonOk / ffmpegOk / browserOk / renderer / llmReady`）。

## 4. 发布前自检清单

```powershell
node --check index.js
node --check client.js
python -m compileall -q engine/python
node scripts/selftest.mjs        # 假 ctx 跑通 health / catalog / jobs
```

再跑一次真实出片（两种模式各一条）：

```powershell
python engine/python/tools/build_job.py <job.json>   # native 与 prompt 各一条
```

CI（`.github/workflows/ci.yml`）只做语法与结构检查，因为 CI 机器上没有浏览器和 ffmpeg。

## 5. GitHub 账号 / 用户名改了之后

用户名改名后 GitHub 会给旧地址留**重定向**（`301`），但有两个前提：账户还在、旧名字没被别人抢注。
所以不要依赖重定向，把地址改到新名字上才算稳。

仓库里要改的就三处（本次 `0.3.2` 已改，可作为模板）：

1. `package.json` 的 `repository.url` / `homepage` / `bugs`
2. `README.md` 的 `git clone` 地址
3. 全文搜索旧名：`git grep -n -i '<旧用户名>'`

本地 git 配置也要跟着改：

```powershell
git remote set-url origin https://github.com/<新用户名>/dsh-ai-fastvideo.git
git config user.name  "<新用户名>"
git config user.email "<数字ID>+<新用户名>@users.noreply.github.com"   # ID 用 gh api user 查
git push                                  # 用新地址推一次，确认凭据没问题
```

查新数字 ID（noreply 邮箱要用它，否则贡献图不计入）：

```powershell
gh api user        # 取 "id" 字段
```

还需要自己在 GitHub 上确认（这些不在仓库里，改名偶尔会悄悄失效）：

- **SSH keys / GPG keys / PAT**：改名一般不影响，但用 `ssh -T git@github.com`、`gh auth status` 各验一次；
  PAT 的 `repo` / `workflow` scope 仍要够推送和跑 Actions。
- **Actions**：仓库名没变就没事；如果顺手改了仓库名，要检查 workflow 里的 `if:` 条件、badge URL、`permissions`。
- **Pages / Webhooks / Deploy keys / 第三方 OAuth 应用**：凡是回填过 `https://github.com/<旧名>/...` 的都要改。
- **其他仓库、个人主页 README、B 站简介、博客、Docker 镜像标签、CI 里的 `ghcr.io/<旧名>/...`**：
  外部引用是最容易漏的，重定向失效后就是死链。
- **npm**：如果已经 `npm publish` 过，`repository` 字段跟着新 `package.json` 再发一版即可；npm 账号名与 GitHub 名无关。
- **旧名字的处置**：不想被陌生人抢注、导致旧链接跳到别人仓库的话，可以注册一个占位小号把旧名字占住；
  不介意的话就什么都不做，接受重定向在旧名被占用后失效。

改完验一遍重定向确实还活着（旧地址应回 `301` 指向新地址）：

```powershell
curl.exe -s -o NUL -w "%{http_code} %{redirect_url}\n" https://github.com/<旧用户名>/dsh-ai-fastvideo
```

