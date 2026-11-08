# 一季一鲜 · 助农电商平台（v2 · 完善版）

> 基于 [v1 小组版](https://github.com/Yunffu/yijiyixian) 的**工程化改造版**。
> v1 保留原始形态存档；本仓库承载缺陷修复、接口契约收敛与可测试性改造。

| | |
|---|---|
| 项目名称 | 一季一鲜 · 助农电商平台（完善版） |
| 项目性质 | **个人主导改造**（原作者接手自己的小组项目做质量与工程化升级） |
| 起始版本 | v1（2026.11 小组版）→ 本仓库 |
| 配套工程 | [`Yunffu/yijiyixian-qa`](https://github.com/Yunffu/yijiyixian-qa) —— 接口自动化测试工程 |
| 仓库 | `Yunffu/yijiyixian-v2` |

---

## 1. 为什么有 v2

v1 是一个**能演示、能交互**的助农电商前端与最小后端。接入真实接口测试后，定位出 4 个可复现的工程问题：

| # | 问题 | 复现方式 | v1 位置 |
|---|---|---|---|
| 1 | `quantity` 无正整数校验，传负数令库存虚增 | `POST /api/orders {"quantity": -5}` → 库存 999 → **1004** | `backend/server.js:329 / 377` |
| 2 | `total_price` 返回单价而非「单价 × 数量」；`price` 字段为 TEXT | `{"quantity": 2}` → `total_price = "¥1288"`（单价） | `backend/server.js:387` |
| 3 | 库存扣减「读-判-写」未 await 且无事务 → 并发超卖 | 12 并发抢 5 件库存 → **12 单全成功、stock = -7** | `backend/server.js:329 / 377` |
| 4 | 商品列表页 `products.html` 仍读 `data.js` 静态常量，前后端数据可能不一致 | 后端改价后列表页不变 | `products.html:416` |

缺陷复现脚本与数据库前后快照见 `yijiyixian-qa` 仓库。

---

## 2. 改造路线

### 阶段 A · 数据与接口正确性
- [ ] `quantity` 正整数校验（非法值返回 400）
- [ ] `price` 改为数值列；`total_price = 单价 × 数量`；订单落**下单时价格快照**
- [ ] 原子扣减：`UPDATE products SET stock = stock - ? WHERE id = ? AND stock >= ?` + 校验影响行数
- [ ] 下单 / 取消订单纳入事务，取消时做金额与库存冲销

### 阶段 B · 前端一致性
- [ ] `products.html` 列表页接入 `ProductAPI.list`，去掉 `mockProducts`
- [ ] `js/api.js` 的 `API_BASE` 改为同源相对路径 `/api`，消除硬编码域名

### 阶段 C · 可测试性与工程化
- [ ] 统一异常码表与错误响应结构
- [ ] 参数校验下沉到统一的校验层
- [ ] 接口日志（含 trace_id）便于定位
- [ ] 数据库索引补充
- [ ] CI：GitHub Actions 跑接口自动化测试

---

## 3. 与 v1 的差异说明

v1 与 v2 的差异**不在页面与视觉**（交互与设计保持一致），而在**后端正确性与工程规范**。
每一条改动都会对应到 `yijiyixian-qa` 仓库里的一条可执行断言。

---

## 4. 本地运行

```bash
cd backend
npm install
npm start          # → http://localhost:3001
```

数据库首次启动自动创建并由 `initProducts()`（`backend/server.js:639`）从 `data.js` 灌入 267 条商品。

---

## 5. 仓库关系

```
yijiyixian      （v1 · 小组项目存档）
      └── yijiyixian-v2   （完善版 · 本仓库，改造进行中）
              └── yijiyixian-qa   （接口自动化测试工程，双 SUT 对照）
```
