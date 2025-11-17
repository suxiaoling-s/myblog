# 欢迎来到我的技术博客

这是我的个人技术博客，记录在开发 **RadarLive-CN** 项目过程中的学习收获、技术总结和项目文档。

## 关于 RadarLive-CN 项目

**RadarLive-CN** 是一个雷达数据实时展示系统，类似中央气象台的雷达数据展示网站。系统支持雷达拼图和单站雷达两种模式，提供实时数据展示、时间轴播放、区域筛选等功能。

### 技术栈

- **后端**: FastAPI + PostgreSQL + SQLAlchemy
- **前端**: Vue.js 3 + Element Plus + Leaflet
- **部署**: Docker + Docker Compose + Nginx

### 核心功能

- ✅ 雷达图像实时展示
- ✅ 时间轴播放控制（播放/暂停/循环）
- ✅ 区域和站点筛选
- ✅ 用户登录认证（JWT Token）
- ✅ 图片缩放和预览
- ✅ 智能数据预加载和缓存

## 博客内容

### 📚 项目文档

完整的项目技术文档，包括：

- [项目结构总览](项目实战/项目结构总览.md) - 项目整体架构和目录结构
- [后端架构详解](项目实战/后端架构详解.md) - FastAPI 后端实现细节
- [前端架构详解](项目实战/前端架构详解.md) - Vue.js 前端实现细节
- [API 接口文档](项目实战/API接口文档.md) - 完整的 API 接口说明

### 📖 学习总结

- [RadarLive-CN 项目学习收获](学习总结/RadarLive-CN项目学习收获.md) - 项目开发过程中的主要收获和心得

## 快速开始

### 本地预览博客

```bash
# 进入博客目录
cd blog

# 安装依赖
pip install -r requirements.txt

# 启动开发服务器
mkdocs serve

# 访问 http://127.0.0.1:8000
```

### 部署到 GitHub Pages

详细步骤请查看 [部署指南](../部署指南.md)。

## 项目链接

- **项目仓库**: [GitHub](https://github.com/yourusername/radarlive-cn) (请替换为实际地址)
- **在线演示**: [Demo](http://your-demo-url.com) (请替换为实际地址)

## 技术亮点

1. **高性能**: 异步 I/O、数据库索引优化、前端图片预加载
2. **用户体验**: 响应式设计、流畅的播放控制、智能数据加载
3. **安全性**: JWT Token 认证、密码加密存储、验证码防刷
4. **可维护性**: 模块化设计、清晰的代码结构、完善的日志记录

---

*最后更新: {{ git_revision_date_localized }}*

