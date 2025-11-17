# 个人技术博客

基于 MkDocs + Material 的简单个人博客，记录 RadarLive-CN 项目学习收获。

## 快速开始

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

### 2. 本地预览

```bash
mkdocs serve
```

访问 http://127.0.0.1:8000

### 3. 编辑内容

编辑 `docs/学习总结/RadarLive-CN项目学习收获.md` 写你的学习收获

## 部署到 GitHub Pages

详细步骤请查看 [部署指南.md](部署指南.md)

简单步骤：
1. 在 GitHub 创建新仓库
2. 修改 `mkdocs.yml` 中的仓库信息
3. 推送代码到 GitHub
4. GitHub Actions 会自动部署（或使用 `mkdocs gh-deploy`）

## 项目结构

```
blog/
├── docs/                    # 文档源文件
│   ├── index.md            # 首页
│   ├── about.md            # 关于页面
│   └── 学习总结/           # 学习总结文章
├── .github/workflows/      # GitHub Actions 自动部署
├── mkdocs.yml              # MkDocs 配置
├── requirements.txt        # Python 依赖
└── README.md              # 本文件
```

## 参考

- [部署指南.md](部署指南.md) - 详细的部署步骤
- [MkDocs 文档](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)

