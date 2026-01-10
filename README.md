# getUserStarsToPage(Github Star Manger)
getUserStarsToPage 可以方便查询和整理你的 Github Star 仓库, 而不必每次使用 Github Star 功能(不太直观且有分页)

[预览](https://get-user-stars-to-page.vercel.app/)

## 使用

- fork 项目
- 配置 Github Token
  - 个人资料下`settings->Developer settings->Personal access tokens->Tokens (classic)` 里设置一个token 过期时间设置永不过期或者其他过期时间。
  - 在当前项目里的`Settings->secrets and variables->Actions` 添加一个secrets, 名称key为`GH_TOKEN`, value就是Tokens (classic)里的token。
- 部署项目到 vercel 或其他平台

## 技术栈

- 前端框架：Nuxt.js
- 数据来源：GitHub API
- 更新机制：每日自动同步 `fetch-star-projects`

## 开发

```bash
# 安装依赖
bun i

# 启动开发服务器
bun run dev

# 运行脚本 同步 Github Star 仓库
bun run sync
```

---
## 原理分析

GitHub Copilot Chat Assistant

结论（简短）：在这个 repo 中，bun run sync 会执行 package.json 里的 "sync" 脚本（即运行 fetch-star-projects.ts）。该脚本用 GitHub Token 调用 Octokit 拉取当前用户的 starred 仓库，生成一个精简的 JSON 数组并写入文件 server/api/all-stars.json；随后 workflow 会把修改的文件 git add/commit 并 push 到仓库。

详细步骤和影响文件：
- bun run sync 的含义
  - package.json 中 "sync": "bun run fetch-star-projects.ts" —— 所以运行 bun run sync 会直接执行 fetch-star-projects.ts。

- fetch-star-projects.ts 做了什么（获取数据的细节）
  - 从环境变量读取 GH_TOKEN： const GITHUB_TOKEN = process.env.GH_TOKEN，然后用 new Octokit({ auth: GITHUB_TOKEN })。
  - 使用分页调用 GitHub API: GET /user/starred，per_page=100，从 page=1 起，每页之间等待 1 秒（setTimeout 1000 ms），直到返回的 response.data.length === 0 为止，从而收集用户所有 starred 仓库。
  - 对每个仓库做映射（copiedStars），只保留这些字段：
    - stargazers_count, language, description, full_name, name, topics
    - user: { avatar: owner.avatar_url, login: owner.login }
    - html_url, created_at, updated_at
  - 最后用 fs.writeFileSync 写入文件：
    - path: server/api/all-stars.json（通过 path.join(__dirname, 'server', 'api', 'all-stars.json')）
    - 内容是 JSON.stringify(copiedStars)（未做 prettify，紧凑的 JSON）

- workflow 中的后续操作（哪些文件会被提交/推送）
  - .github/workflows/fetch-star-projects.yml 在运行脚本后执行：
    - git add .
    - git commit -m "chore: fetch star projects [skip ci]" || exit 0
    - git push
  - 因此，脚本生成或更新的文件会被提交并推送。最确定会被更改/提交的文件是:
    - server/api/all-stars.json（由脚本写入/更新）
  - 另外注意：
    - bun install 可能在工作区产生 bun.lockb 或 node_modules（取决于 Bun 行为）；如果这些文件在工作区且被修改，也会被包含在 git add . 中。实际会提交哪些文件以 git status/gpi add 前的状态为准，但脚本本身只显式写入 server/api/all-stars.json。
