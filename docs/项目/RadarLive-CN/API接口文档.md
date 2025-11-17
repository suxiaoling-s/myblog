# API 接口文档

## 基础信息

- **Base URL**: `http://localhost:8000` (开发环境)
- **API 前缀**: `/api`
- **认证方式**: Bearer Token (JWT)

## 认证接口

### 1. 用户登录

**POST** `/api/login`

**请求体**:
```json
{
  "username": "string",
  "password": "string"
}
```

**响应**:
```json
{
  "access_token": "string",
  "refresh_token": "string",
  "token_type": "bearer",
  "username": "string"
}
```

**说明**:
- access_token 有效期: 30分钟
- refresh_token 有效期: 7天
- 需要在请求头中携带: `Authorization: Bearer {access_token}`

### 2. 用户注册

**POST** `/api/register`

**请求体**:
```json
{
  "username": "string",      // 3-50字符，字母数字下划线连字符
  "password": "string",      // 至少6字符
  "captcha_id": "string",
  "captcha_text": "string"
}
```

**响应**:
```json
{
  "message": "注册成功",
  "username": "string"
}
```

### 3. 获取验证码

**GET** `/api/captcha`

**响应**:
```json
{
  "captcha_id": "string"
}
```

**获取验证码图片**: `GET /api/captcha/{captcha_id}/image`

### 4. 修改密码

**POST** `/api/change-password` (需要认证)

**请求体**:
```json
{
  "current_password": "string",
  "new_password": "string"    // 至少6字符
}
```

**响应**:
```json
{
  "message": "密码已更新"
}
```

### 5. 刷新 Token

**POST** `/api/refresh`

**请求体**:
```json
{
  "refresh_token": "string"
}
```

**响应**:
```json
{
  "access_token": "string",
  "token_type": "bearer"
}
```

## 雷达拼图接口

所有接口都需要认证（Bearer Token）。

### 1. 获取最新雷达图像

**GET** `/api/radar/latest`

**响应**:
```json
{
  "filename": "string",
  "timestamp": "2024-01-01T12:00:00Z",
  "url": "http://...",
  "size": 123456,
  "region": "EAST"
}
```

### 2. 获取雷达图像列表

**GET** `/api/radar/list`

**查询参数**:
- `start_time` (可选): ISO 格式开始时间
- `end_time` (可选): ISO 格式结束时间
- `limit` (可选, 默认5000): 最大返回数量 (1-5000)
- `region` (可选): 区域代码 (EAST, NORTH, etc.)
- `page` (可选): 页码 (1-based)
- `page_size` (可选): 每页数量 (1-200)

**响应**:
```json
{
  "total": 100,
  "images": [
    {
      "filename": "string",
      "timestamp": "2024-01-01T12:00:00Z",
      "url": "http://...",
      "size": 123456,
      "region": "EAST"
    }
  ]
}
```

### 3. 获取时间轴数据

**GET** `/api/radar/timeline`

**查询参数**:
- `hours` (可选, 默认24): 查询最近N小时 (1-168)
- `region` (可选): 区域代码

**响应**:
```json
{
  "start_time": "2024-01-01T00:00:00Z",
  "end_time": "2024-01-01T12:00:00Z",
  "count": 100,
  "images": [...]
}
```

### 4. 获取数据时间范围

**GET** `/api/radar/time-range`

**查询参数**:
- `region` (可选): 区域代码

**响应**:
```json
{
  "earliest": "2024-01-01T00:00:00Z",
  "latest": "2024-01-01T12:00:00Z",
  "count": 1000
}
```

### 5. 快速获取区域最新数据

**GET** `/api/radar/latest-by-region`

**查询参数**:
- `region` (必需): 区域代码

**响应**:
```json
{
  "status": "success",
  "image": {
    "filename": "string",
    "timestamp": "2024-01-01T12:00:00Z",
    "url": "http://...",
    "size": 123456,
    "region": "EAST"
  }
}
```

## 单站雷达接口

所有接口都需要认证（Bearer Token）。

### 1. 获取单站雷达站点列表

**GET** `/api/radar/single/stations`

**查询参数**:
- `province` (可选): 省份名称
- `city` (可选): 城市名称

**响应**:
```json
[
  {
    "station_code": "Z9000",
    "province": "北京",
    "city": "北京",
    "latest_timestamp": "2024-01-01T12:00:00Z",
    "image_count": 1000
  }
]
```

### 2. 获取单站雷达图像列表

**GET** `/api/radar/single/list`

**查询参数**:
- `station_code` (必需): 站点编码
- `start_time` (可选): ISO 格式开始时间
- `end_time` (可选): ISO 格式结束时间
- `limit` (可选, 默认500): 最大返回数量 (1-5000)
- `province` (可选): 省份过滤
- `city` (可选): 城市过滤
- `page` (可选): 页码
- `page_size` (可选): 每页数量

**响应**:
```json
{
  "total": 100,
  "images": [
    {
      "filename": "string",
      "timestamp": "2024-01-01T12:00:00Z",
      "url": "http://...",
      "size": 123456,
      "station_code": "Z9000",
      "province": "北京",
      "city": "北京"
    }
  ]
}
```

### 3. 获取单站最新图像

**GET** `/api/radar/single/latest`

**查询参数**:
- `station_code` (必需): 站点编码

**响应**:
```json
{
  "filename": "string",
  "timestamp": "2024-01-01T12:00:00Z",
  "url": "http://...",
  "size": 123456,
  "station_code": "Z9000",
  "province": "北京",
  "city": "北京"
}
```

### 4. 获取单站数据时间范围

**GET** `/api/radar/single/time-range`

**查询参数**:
- `station_code` (必需): 站点编码
- `province` (可选): 省份过滤
- `city` (可选): 城市过滤

**响应**:
```json
{
  "earliest": "2024-01-01T00:00:00Z",
  "latest": "2024-01-01T12:00:00Z",
  "count": 1000
}
```

## 错误响应

所有接口在出错时返回标准错误格式：

```json
{
  "detail": "错误描述信息"
}
```

**HTTP 状态码**:
- `200`: 成功
- `400`: 请求参数错误
- `401`: 未认证或 Token 无效
- `404`: 资源不存在
- `500`: 服务器内部错误

## 使用示例

### JavaScript (Axios)

```javascript
import axios from 'axios';

// 设置基础 URL
const api = axios.create({
  baseURL: 'http://localhost:8000/api'
});

// 登录
const login = async (username, password) => {
  const response = await api.post('/login', { username, password });
  const { access_token } = response.data;
  
  // 存储 token
  localStorage.setItem('auth_token', access_token);
  
  // 设置默认请求头
  api.defaults.headers.common['Authorization'] = `Bearer ${access_token}`;
  
  return response.data;
};

// 获取雷达列表
const getRadarList = async (region = null) => {
  const params = region ? { region } : {};
  const response = await api.get('/radar/list', { params });
  return response.data;
};
```

### Python (requests)

```python
import requests

BASE_URL = "http://localhost:8000/api"

# 登录
def login(username, password):
    response = requests.post(
        f"{BASE_URL}/login",
        json={"username": username, "password": password}
    )
    data = response.json()
    token = data["access_token"]
    return token

# 获取雷达列表
def get_radar_list(token, region=None):
    headers = {"Authorization": f"Bearer {token}"}
    params = {"region": region} if region else {}
    response = requests.get(
        f"{BASE_URL}/radar/list",
        headers=headers,
        params=params
    )
    return response.json()
```

## 注意事项

1. **Token 过期**: access_token 过期后需要使用 refresh_token 刷新
2. **时间格式**: 所有时间使用 ISO 8601 格式，UTC 时区
3. **分页**: 大数据量查询建议使用分页参数
4. **限流**: （可扩展）建议实现请求限流
5. **缓存**: 客户端可以缓存静态数据，减少请求

