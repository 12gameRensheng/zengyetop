# zengyetop

基于 **Vite + Vue 3** 的导航式学习博客，构建时会自动编译 `src/posts` 目录下的 Markdown 文件为静态页面。

## 使用

```bash
npm install
npm run dev    # 本地预览
npm run build  # 生成 dist 纯静态文件
```

## 添加内容

- 在 `src/posts` 下新增或修改 `.md` 文件即可出现在左侧导航。
- 可以使用子目录区分分类，例如 `src/posts/vue/xxx.md`，分类名取自子目录名称。

## 部署

`npm run build` 后的 `dist` 目录可直接托管到 GitHub Pages、Vercel 或任意静态服务器。
