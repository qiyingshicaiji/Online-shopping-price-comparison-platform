# 《一次买够》网购比价平台

> 依据《项目需求规格说明书》和 `openapi.json` 提供的接口契约，完成的后端示例实现（FastAPI）与文档补充。

## 背景与核心需求

来自需求文档的关键要点：

- **核心价值**：一次搜索聚合淘宝、京东、拼多多等平台价格与促销，降低比价成本，并提供历史趋势与跳转购买入口。
- **用户故事覆盖**：  
  - 比价者：同屏比价多平台同款商品。  
  - 精打细算者：按价格、销量、平台、信誉排序筛选。  
  - 观望者：收藏并设置降价提醒。  
  - 决策者：查看 90 天价格趋势，判断是否入手。  
  - 行动者：一键跳转原平台完成购买。
- **主要功能**：商品聚合搜索/筛选、商品详情与跳转、微信登录、收藏与降价提醒、浏览历史与搜索记录、个性化推荐与埋点上报。
- **非功能性提示**：模块化解耦、基础安全防护、请求性能目标（常规搜索 < 2s，百级并发）、容器化与 CI/CD（文档推荐 Django/MySQL/Redis，可按场景替换）。

## 当前后端实现概述

本仓库包含一个 **FastAPI** 演示服务，使用内存数据模拟商品、收藏、降价提醒等场景，并完整覆盖 `openapi.json` 中定义的 16 个接口（含鉴权流程）。适合前端/联调/演示使用。

- 语言/框架：Python 3.12 + FastAPI + Uvicorn
- 认证方式：Bearer Token（登录返回 access_token / refresh_token）
- 数据存储：内存模拟数据（无外部依赖，重启即重置）
- API 契约：与根目录 `openapi.json` 保持一致

## 快速开始

```bash
# 1) 安装依赖
pip install -r requirements.txt

# 2) 启动服务 (默认 http://127.0.0.1:8000)
uvicorn app.main:app --reload

# 3) 打开交互文档
# http://127.0.0.1:8000/docs 或 /redoc
```

> 如果需要自测，可运行：`python -m pytest`

## 主要接口速览

所有接口的入参/出参请以 `openapi.json` 为准，以下为常用示例：

### 认证
- `POST /auth/login`：传入 `{code, raw_data, signature}` 获取 `access_token`、`refresh_token`。  
- `POST /auth/refresh`：使用 `refresh_token` 换新。  
- `POST /auth/logout`：可选携带 `Authorization: Bearer <token>` 让服务端清理。

> 需鉴权的接口请在 Header 添加：`Authorization: Bearer <access_token>`

### 商品与搜索
- `GET /products/search`：关键词搜索，支持价格区间、类目、平台过滤及排序（price_asc/price_desc/sales_desc/rating_desc），返回分页列表。  
- `GET /products/{product_id}`：获取商品详情（含多平台价格）。  
- `GET /products/categories`：按 `parent_id` 获取分类树。

### 用户与收藏
- `GET/PUT /users/profile`：查询/更新用户基础信息。  
- `GET/POST /users/favorites`：分页查看与新增收藏；`PUT/DELETE /users/favorites/{favorite_id}` 更新备注或删除。

### 降价提醒
- `GET/POST /users/price-alerts`：分页查询或创建提醒（可选监控平台）。  
- `PUT/DELETE /users/price-alerts/{alert_id}`：更新目标价/启用状态或删除。

### 浏览历史 & 搜索
- `GET/POST/DELETE /users/browse-history`：分页获取、追加或清空浏览记录。  
- `GET /users/search-records`：分页返回搜索历史。  
- `GET /search/hot-words`：返回热搜词榜（公开接口）。

### 推荐与埋点
- `GET /recommendations/personalized`：基于行为的个性化推荐（示例数据）。  
- `POST /analytics/events`：上报用户事件（search、product_click、add_favorite 等）。

## 设计与实现说明

- **统一响应格式**：`ApiResponse` `{code, message, data}`，成功 `code=0`，错误返回非 0 业务码与 HTTP 状态码。  
- **分页**：采用 `page/page_size`，返回 `list + pagination{page,page_size,total,total_pages,has_next,has_prev}`。  
- **鉴权**：除登录/刷新/热搜等公开接口外，其余均需 Bearer Token。示例 Token 为内存颁发，适合联调与演示。  
- **示例数据**：商品/平台报价、收藏、降价提醒、历史记录、热搜词均为内存示例，重启后重置，可按需扩展或替换为数据库/爬虫数据源。

## 开发与扩展建议

- 若接入真实数据，可按需求文档推荐的分层方式将数据层替换为数据库（MySQL/Redis）及爬虫采集模块。  
- 安全与风控：生产环境需替换为真实 JWT、鉴权中间件，并完善速率限制、输入校验与监控告警。  
- 部署：可使用 `uvicorn` + `gunicorn` 或容器化部署，前置 Nginx/负载均衡，与 CI/CD 流程集成。  

## 参考资料

- 需求文档：`项目需求规格说明书.docx`  
- 接口契约：`openapi.json`（OpenAPI 3.0）
