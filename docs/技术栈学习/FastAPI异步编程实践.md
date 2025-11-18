# FastAPI 异步编程实践

## 前言

在开发 RadarLive-CN 项目时，我深入学习了 FastAPI 的异步编程模式。通过使用 `async/await` 语法和异步数据库驱动，显著提升了应用的并发处理能力。本文分享我在实际项目中的使用经验和性能优化心得。

## 什么是异步编程

异步编程允许程序在等待 I/O 操作（如数据库查询、网络请求）时继续处理其他任务，而不是阻塞等待。这对于高并发的 Web 应用非常重要。

## FastAPI 异步基础

### 基本语法

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/radar/latest")
async def get_latest_radar():
    """异步 API 端点"""
    scanner = get_scanner()
    latest = await scanner.get_latest_image()  # 异步数据库查询
    return latest
```

### 异步数据库操作

使用 `asyncpg` 作为 PostgreSQL 的异步驱动：

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

# 创建异步引擎
engine = create_async_engine(
    "postgresql+asyncpg://user:password@localhost/dbname",
    pool_pre_ping=True,
    pool_size=10,
    max_overflow=20
)

# 创建异步会话工厂
AsyncSessionLocal = sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False
)

# 在路由中使用
@app.get("/radar/latest")
async def get_latest_radar():
    async with AsyncSessionLocal() as session:
        stmt = select(RadarImageModel).order_by(RadarImageModel.timestamp.desc())
        result = await session.execute(stmt)
        latest = result.scalar_one_or_none()
        return latest
```

## 实际项目案例

### 完整的异步 API 实现

```python
from fastapi import FastAPI, HTTPException, status
from sqlalchemy.exc import SQLAlchemyError
from loguru import logger

@radar_router.get("/radar/latest")
async def get_latest_radar():
    """获取最新雷达图像（异步）"""
    try:
        scanner = get_scanner()
        latest = await scanner.get_latest_image()  # 异步数据库查询
        if not latest:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="No radar images found"
            )
        return latest
    except SQLAlchemyError as e:
        logger.error(f"Database error: {str(e)}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Database error: {str(e)}"
        )
```

### 异步业务逻辑服务

```python
class RadarScanner:
    async def get_latest_image(self):
        """异步获取最新图像"""
        async with AsyncSessionLocal() as session:
            stmt = (
                select(RadarImageModel)
                .order_by(RadarImageModel.timestamp.desc())
                .limit(1)
            )
            result = await session.execute(stmt)
            image = result.scalar_one_or_none()
            
            if image:
                return self._image_to_dict(image)
            return None
    
    async def get_image_list(self, start_time=None, end_time=None, 
                            region=None, page=1, page_size=100):
        """异步获取图像列表（支持分页和筛选）"""
        async with AsyncSessionLocal() as session:
            stmt = select(RadarImageModel)
            
            # 添加筛选条件
            if start_time:
                stmt = stmt.where(RadarImageModel.timestamp >= start_time)
            if end_time:
                stmt = stmt.where(RadarImageModel.timestamp <= end_time)
            if region:
                stmt = stmt.where(RadarImageModel.region == region)
            
            # 分页
            stmt = stmt.order_by(RadarImageModel.timestamp.desc())
            stmt = stmt.offset((page - 1) * page_size).limit(page_size)
            
            result = await session.execute(stmt)
            images = result.scalars().all()
            
            return [self._image_to_dict(img) for img in images]
```

## 性能优化

### 连接池配置

```python
engine = create_async_engine(
    DATABASE_URL,
    pool_pre_ping=True,      # 连接前检查连接是否有效
    pool_size=10,            # 连接池大小
    max_overflow=20,         # 最大溢出连接数
    pool_recycle=3600        # 连接回收时间（秒）
)
```

### 性能提升数据

在 RadarLive-CN 项目中，使用异步 API 后：

- **并发处理能力**：相比同步版本，可以同时处理更多并发请求
- **响应时间**：整体响应时间降低约 30-40%
- **资源利用**：数据库查询不会阻塞其他请求的处理

## 实践心得

### 1. 异步上下文管理

- 使用 `async with` 确保数据库会话正确关闭
- 避免在异步函数中使用同步数据库操作

### 2. 错误处理

- 捕获 `SQLAlchemyError` 等数据库异常
- 记录详细错误日志
- 返回友好的错误信息给客户端

### 3. 连接池配置

- 根据实际并发量调整 `pool_size` 和 `max_overflow`
- 使用 `pool_pre_ping` 检测连接有效性
- 设置合理的 `pool_recycle` 时间

### 4. 性能监控

- 监控数据库查询时间
- 监控连接池使用情况
- 根据实际数据调整配置

## 总结

FastAPI 的异步编程模式让应用能够高效处理并发请求。通过：

- ✅ 使用 `async/await` 语法
- ✅ 使用异步数据库驱动（asyncpg）
- ✅ 合理配置连接池
- ✅ 正确处理错误和异常

我们可以构建出高性能、高并发的 Web API 服务。

---

*本文基于 RadarLive-CN 项目的实际开发经验总结*

