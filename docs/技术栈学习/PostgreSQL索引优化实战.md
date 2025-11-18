# PostgreSQL 索引优化实战

## 前言

在开发 RadarLive-CN 项目时，我深入学习了 PostgreSQL 的索引优化。通过合理设计索引，显著提升了数据库查询性能。本文分享我在实际项目中的索引设计经验和优化心得。

## 索引基础

### 什么是索引

索引是数据库中用于快速查找数据的数据结构，类似于书籍的目录。

### 索引类型

- **B-tree 索引**：最常用的索引类型，适合等值查询和范围查询
- **复合索引**：多个列组合的索引
- **部分索引**：只对满足条件的行建立索引

## 项目中的索引设计

### 雷达拼图图像表索引

```python
class RadarImageModel(Base):
    __tablename__ = "radar_images"
    
    id = Column(Integer, primary_key=True)
    filename = Column(String(255), unique=True, nullable=False, index=True)
    timestamp = Column(DateTime(timezone=True), nullable=False, index=True)
    region = Column(String(50), nullable=True, index=True)
    
    # 复合索引
    __table_args__ = (
        Index('idx_timestamp_filename', 'timestamp', 'filename'),
        Index('idx_region_timestamp', 'region', 'timestamp'),
        Index('idx_region_timestamp_desc', 'region', 'timestamp', 
              postgresql_ops={'timestamp': 'DESC'}),
    )
```

### 单站雷达图像表索引

```python
class SingleRadarImageModel(Base):
    __tablename__ = "single_radar_images"
    
    station_code = Column(String(50), nullable=False, index=True)
    timestamp = Column(DateTime(timezone=True), nullable=False, index=True)
    province = Column(String(100), nullable=True, index=True)
    city = Column(String(100), nullable=True, index=True)
    
    __table_args__ = (
        Index('idx_single_station_timestamp', 'station_code', 'timestamp'),
        Index('idx_single_province_city', 'province', 'city'),
    )
```

## 索引设计原则

### 1. 根据查询模式设计

- 经常用于 WHERE 条件的列 → 建立索引
- 经常用于 ORDER BY 的列 → 建立索引
- 经常用于 JOIN 的列 → 建立索引

### 2. 复合索引的列顺序

- 最常用的列放在前面
- 选择性高的列放在前面
- 考虑查询模式

### 3. 避免过度索引

- 索引会占用存储空间
- 索引会降低写入性能
- 只对必要的列建立索引

## 性能优化效果

通过合理的索引设计：

- ✅ 查询速度提升 10-100 倍
- ✅ 排序操作更快
- ✅ 减少全表扫描

## 实践心得

### 1. 分析查询模式

- 使用 `EXPLAIN ANALYZE` 分析查询计划
- 识别慢查询
- 根据实际查询优化索引

### 2. 监控索引使用

- 定期检查索引使用情况
- 删除未使用的索引
- 优化索引结构

### 3. 测试和验证

- 在测试环境验证索引效果
- 对比优化前后的性能
- 持续优化

## 总结

合理的索引设计是数据库性能优化的关键：

- ✅ 根据查询模式设计索引
- ✅ 合理使用复合索引
- ✅ 避免过度索引
- ✅ 持续监控和优化

---

*本文基于 RadarLive-CN 项目的实际开发经验总结*

