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
