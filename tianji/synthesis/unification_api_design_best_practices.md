# 玄学API设计的最佳实践统一分析

## 一、概述

本文档对集群E审计的数据与API服务（农历API服务、节气计算微服务、JPL星历表API、八字数据库）进行API设计最佳实践的统一分析，提取API设计模式、接口规范和安全性建议。

## 二、API架构模式对比

### 2.1 架构风格分布

**RESTful API（农历API服务、节气微服务）**
- 资源导向设计
- HTTP方法语义化
- JSON数据格式
- 状态码规范

**GraphQL（部分现代服务）**
- 灵活查询
- 强类型Schema
- 单一端点
- 减少过度获取

**RPC风格（JPL Horizons）**
- 过程调用导向
- 参数传递
- 特定领域协议

### 2.2 端点设计对比

**农历API服务（RESTful）**
```
GET /api/holidays              # 获取节假日列表
GET /api/holidays/:year        # 获取指定年份节假日
GET /api/solar-terms/:year     # 获取节气
POST /api/convert/solar-to-lunar  # 公历转农历
```

**节气微服务（RESTful）**
```
GET /lunar/:date               # 获取农历信息
POST /lunar/batch              # 批量查询
GET /health                    # 健康检查
```

**JPL Horizons（类RPC）**
```
GET /api/horizons.api?COMMAND=399&...  # 查询天体数据
```

## 三、接口设计最佳实践

### 3.1 URL设计原则

**资源命名**
```
✓ /api/bazi/charts             # 复数形式
✓ /api/bazi/charts/:id         # 资源ID
✗ /api/getBaziChart            # 避免动词
✗ /api/bazi_chart              # 避免下划线
```

**层次结构**
```
/api/v1/bazi/charts            # 版本控制
/api/v1/bazi/charts/:id/dayun  # 子资源
/api/v1/bazi/charts/:id/liunian?year=2024  # 查询参数
```

**HTTP方法语义**
```
GET    /api/bazi/charts        # 列表查询
POST   /api/bazi/charts        # 创建资源
GET    /api/bazi/charts/:id    # 获取详情
PUT    /api/bazi/charts/:id    # 全量更新
PATCH  /api/bazi/charts/:id    # 部分更新
DELETE /api/bazi/charts/:id    # 删除资源
```

### 3.2 请求/响应格式

**请求格式**
```typescript
// POST /api/bazi/charts
{
  "birth_date": "1990-01-01",
  "birth_hour": 12,
  "gender": "male",
  "options": {
    "show_da_yun": true,
    "show_shen_sha": true
  }
}
```

**响应格式（成功）**
```typescript
{
  "code": 200,
  "message": "success",
  "data": {
    "id": "chart_123",
    "year_pillar": "庚午",
    "month_pillar": "丁丑",
    "day_pillar": "甲子",
    "hour_pillar": "庚午",
    "day_master": "甲",
    "shi_shen": { /* ... */ },
    "da_yun": [ /* ... */ ]
  },
  "meta": {
    "request_id": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

**响应格式（错误）**
```typescript
{
  "code": 400,
  "message": "Invalid birth date format",
  "error": {
    "type": "ValidationError",
    "field": "birth_date",
    "details": "Expected format: YYYY-MM-DD"
  },
  "meta": {
    "request_id": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

### 3.3 状态码使用

**2xx 成功**
```
200 OK              # 请求成功
201 Created         # 资源创建成功
204 No Content      # 删除成功
```

**4xx 客户端错误**
```
400 Bad Request     # 请求参数错误
401 Unauthorized    # 未认证
403 Forbidden       # 无权限
404 Not Found       # 资源不存在
422 Unprocessable   # 语义错误（如无效日期）
429 Too Many Requests # 限流
```

**5xx 服务端错误**
```
500 Internal Error  # 服务器内部错误
502 Bad Gateway     # 网关错误
503 Service Unavailable # 服务不可用
504 Gateway Timeout # 超时
```

## 四、API安全最佳实践

### 4.1 认证授权

**API Key认证**
```http
GET /api/bazi/charts HTTP/1.1
Host: api.example.com
X-API-Key: your_api_key_here
```

**JWT Token认证**
```http
GET /api/bazi/charts HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**OAuth 2.0（第三方集成）**
```http
GET /api/bazi/charts HTTP/1.1
Host: api.example.com
Authorization: Bearer {access_token}
```

### 4.2 限流策略

**固定窗口限流**
```typescript
// 每分钟最多100次请求
const rateLimiter = new RateLimiter({
  windowMs: 60 * 1000,
  max: 100
});
```

**滑动窗口限流**
```typescript
// 更平滑的限流策略
const rateLimiter = new SlidingWindowRateLimiter({
  windowMs: 60 * 1000,
  max: 100
});
```

**响应头**
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
```

### 4.3 输入验证

**参数验证**
```typescript
// 使用Joi或Zod进行验证
const schema = z.object({
  birth_date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
  birth_hour: z.number().int().min(0).max(23),
  gender: z.enum(['male', 'female'])
});

// 验证请求
const result = schema.safeParse(req.body);
if (!result.success) {
  return res.status(400).json({
    code: 400,
    message: 'Validation failed',
    errors: result.error.errors
  });
}
```

**SQL注入防护**
```typescript
// 使用参数化查询
const query = 'SELECT * FROM charts WHERE id = ?';
db.query(query, [chartId]);

// 避免字符串拼接
// ✗ const query = `SELECT * FROM charts WHERE id = ${chartId}`;
```

## 五、性能优化最佳实践

### 5.1 缓存策略

**HTTP缓存**
```http
Cache-Control: public, max-age=3600  # 缓存1小时
ETag: "abc123"                       # 实体标签
Last-Modified: Mon, 01 Jan 2024 00:00:00 GMT
```

**应用缓存**
```typescript
// Redis缓存
const cacheKey = `bazi:${date}:${hour}:${gender}`;
const cached = await redis.get(cacheKey);

if (cached) {
  return JSON.parse(cached);
}

const result = await calculateBazi(data);
await redis.setex(cacheKey, 3600, JSON.stringify(result));
return result;
```

**CDN缓存**
```
静态数据（节气表）→ CDN缓存
动态数据（排盘结果）→ 应用缓存
```

### 5.2 数据库优化

**索引设计**
```sql
-- 八字查询索引
CREATE INDEX idx_bazi_date ON charts(birth_date, birth_hour);
CREATE INDEX idx_bazi_ganzhi ON charts(day_pillar);

-- 节气查询索引
CREATE INDEX idx_solarterm_date ON solar_terms(date);
```

**查询优化**
```typescript
// 避免N+1查询
// ✗ 循环中逐个查询
for (const id of ids) {
  await db.query('SELECT * FROM charts WHERE id = ?', [id]);
}

// ✓ 批量查询
await db.query('SELECT * FROM charts WHERE id IN (?)', [ids]);
```

### 5.3 异步处理

**后台任务**
```typescript
// 复杂计算放入队列
await queue.add('calculate-bazi', {
  chartId: 'chart_123',
  data: birthData
});

// 立即返回任务ID
return {
  code: 202,
  message: 'Processing',
  data: { jobId: 'job_abc123' }
};
```

**WebSocket实时推送**
```typescript
// 长任务完成后推送
ws.on('connect', (socket) => {
  socket.on('subscribe', (jobId) => {
    socket.join(jobId);
  });
});

// 任务完成时
io.to(jobId).emit('complete', { result });
```

## 六、API版本管理

### 6.1 版本策略

**URL版本**
```
/api/v1/bazi/charts
/api/v2/bazi/charts
```

**Header版本**
```http
GET /api/bazi/charts HTTP/1.1
Accept-Version: v1
```

**兼容性原则**
- 新增字段：向后兼容
- 删除字段：需要版本升级
- 修改字段：需要版本升级

### 6.2 弃用策略

**弃用通知**
```http
Deprecation: Sun, 01 Jan 2024 00:00:00 GMT
Sunset: Sun, 01 Jul 2024 00:00:00 GMT
Link: </api/v2/bazi/charts>; rel="successor-version"
```

**文档更新**
- 明确标注弃用接口
- 提供迁移指南
- 设置弃用时间表

## 七、监控与日志

### 7.1 监控指标

**关键指标**
```
- 请求量（QPS）
- 响应时间（P50, P95, P99）
- 错误率
- 缓存命中率
```

**业务指标**
```
- 排盘成功率
- 节气计算准确率
- 用户活跃度
```

### 7.2 日志规范

**日志格式**
```json
{
  "timestamp": "2024-01-01T00:00:00Z",
  "level": "INFO",
  "request_id": "req_abc123",
  "method": "POST",
  "path": "/api/bazi/charts",
  "status": 200,
  "duration_ms": 45,
  "client_ip": "192.168.1.1",
  "user_agent": "Mozilla/5.0..."
}
```

**日志级别**
```
ERROR: 系统错误，需要立即处理
WARN:  警告，需要注意
INFO:  正常信息
DEBUG: 调试信息
```

## 八、文档与SDK

### 8.1 API文档

**OpenAPI规范**
```yaml
openapi: 3.0.0
info:
  title: 玄学API
  version: 1.0.0
paths:
  /api/bazi/charts:
    post:
      summary: 创建八字排盘
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/BaziRequest'
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/BaziResponse'
```

**文档工具**
- Swagger UI
- ReDoc
- Postman Collection

### 8.2 SDK开发

**JavaScript SDK**
```typescript
class BaziAPI {
  constructor(apiKey: string) {
    this.client = axios.create({
      baseURL: 'https://api.example.com',
      headers: { 'X-API-Key': apiKey }
    });
  }
  
  async calculateChart(data: BirthData): Promise<BaziChart> {
    const response = await this.client.post('/api/bazi/charts', data);
    return response.data;
  }
}

// 使用
const api = new BaziAPI('your_api_key');
const chart = await api.calculateChart({
  birth_date: '1990-01-01',
  birth_hour: 12,
  gender: 'male'
});
```

## 九、总结

玄学API设计的核心原则：

**设计原则**
- RESTful设计，资源导向
- 语义化HTTP方法
- 一致的响应格式
- 完善的错误处理

**安全原则**
- 认证授权机制
- 限流防滥用
- 输入验证
- 数据保护

**性能原则**
- 多级缓存
- 数据库优化
- 异步处理
- 监控告警

**维护原则**
- 版本管理
- 文档完善
- SDK支持
- 日志规范

通过遵循这些最佳实践，可以构建稳定、安全、高性能的玄学API服务。
