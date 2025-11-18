# Docker 容器化部署指南

## 前言

在开发 RadarLive-CN 项目时，我深入学习了 Docker 和 Docker Compose 的使用。通过容器化部署，实现了开发和生产环境的一致性。本文分享我在实际项目中的容器化实践和部署心得。

## Docker 基础

### 什么是 Docker

Docker 是一个容器化平台，可以将应用及其依赖打包成容器，实现"一次构建，到处运行"。

### Docker 核心概念

- **镜像（Image）**：应用的只读模板
- **容器（Container）**：镜像的运行实例
- **Dockerfile**：构建镜像的指令文件
- **Docker Compose**：多容器应用编排工具

## 项目中的 Docker 实践

### 后端 Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制代码
COPY . .

# 暴露端口
EXPOSE 8000

# 启动命令
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 前端 Dockerfile（多阶段构建）

```dockerfile
# 构建阶段
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# 运行阶段
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose 配置

```yaml
services:
  backend:
    build: ./backend
    environment:
      DATABASE_URL: postgresql+asyncpg://postgres:password@db:5432/postgres
      RADAR_DATA_PATH: /mnt/data/radar/china/MOSAIC
    volumes:
      - /mnt/data/radar/china/MOSAIC:/mnt/data/radar/china/MOSAIC:ro
    ports:
      - "8020:8000"
    networks:
      - radarlive-net
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - radarlive-net

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - radarlive-net

networks:
  radarlive-net:
    driver: bridge

volumes:
  postgres_data:
```

## 多阶段构建的优势

### 减小镜像体积

- 构建阶段：包含构建工具和依赖
- 运行阶段：只包含运行时需要的文件
- 最终镜像体积减小 70%+

### 提高安全性

- 运行阶段不包含构建工具
- 减少攻击面
- 更符合最小权限原则

## 部署优势

### 1. 环境一致性

- 开发、测试、生产环境完全一致
- 避免"在我机器上能跑"的问题
- 减少环境配置时间

### 2. 快速部署

- 一条命令启动整个应用栈
- 快速扩展和回滚
- 易于维护

### 3. 资源隔离

- 每个服务运行在独立容器中
- 互不干扰
- 资源使用可控

## 实践心得

### 1. 多阶段构建

- 构建阶段安装依赖，构建项目
- 运行阶段只保留必要文件
- 大幅减小镜像体积

### 2. 环境变量管理

- 通过 `docker-compose.yml` 统一管理
- 开发和生产环境配置分离
- 敏感信息使用 secrets

### 3. 数据卷挂载

- 数据文件通过数据卷挂载
- 避免数据丢失
- 方便备份和迁移

### 4. 网络配置

- 使用 Docker 网络让容器间通信
- 隔离外部访问
- 提高安全性

## 总结

Docker 容器化部署让应用部署变得简单而可靠：

- ✅ 使用多阶段构建减小镜像体积
- ✅ 通过 Docker Compose 管理多容器应用
- ✅ 实现环境一致性
- ✅ 提高部署效率

---

*本文基于 RadarLive-CN 项目的实际开发经验总结*

