---
AIGC:
  ContentProducer: '001191110102MAD55U9H0F10002'
  ContentPropagator: '001191110102MAD55U9H0F10002'
  Label: '1'
  ProduceID: 'db32d57c-0b28-4d89-a8e3-bb68b7995291'
  PropagateID: 'db32d57c-0b28-4d89-a8e3-bb68b7995291'
  ReservedCode1: '6d8de11e-dfe6-4983-9137-3f5adc36d748'
  ReservedCode2: '6d8de11e-dfe6-4983-9137-3f5adc36d748'
---

# BMS管理系统 — 前后端API接口约定文档

> 本文档供**后端开发**参考，定义了前端期望的所有 REST API。前端已内置 Mock 层（`USE_MOCK=true`），后端接口就绪后，将 `src/utils/request.js` 中 `USE_MOCK` 改为 `false` 即可对接真实接口。

---

## 1. 通用约定

### 1.1 基础信息
| 项 | 说明 |
|----|------|
| Base URL | `/api` |
| 协议 | HTTPS（生产）/ HTTP（开发） |
| 数据格式 | JSON（UTF-8） |
| 认证 | 请求头 `Authorization: Bearer {token}` |
| 分页参数 | `page`（页码，从1起）、`pageSize`（每页条数） |

### 1.2 统一响应格式
```json
{
  "code": 0,
  "message": "success",
  "data": {}
}
```
- `code = 0`：成功，前端取 `data`
- `code != 0`：业务错误，前端弹出 `message`
- **HTTP 401**：登录失效，前端自动跳转登录页

### 1.3 分页返回格式
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [],
    "total": 10854
  }
}
```

### 1.4 通用筛选参数
- 时间范围统一格式：`startTime`、`endTime`（`yyyy-MM-dd HH:mm:ss`）
- 日期：`yyyy-MM-dd`
- 布尔值：`true/false`

---

## 2. 认证接口

### 2.1 用户登录
- **POST** `/api/auth/login`
- 请求体：
```json
{ "username": "admin", "password": "******" }
```
- 成功响应 data：
```json
{ "token": "jwt-token-string", "username": "admin", "role": "平台管理员" }
```

### 2.2 退出登录
- **POST** `/api/auth/logout`

---

## 3. 仪表盘接口

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/dashboard/summary | 5张统计卡片 |
| GET | /api/dashboard/activity-trend | 日活跃量趋势（面积图） |
| GET | /api/dashboard/alarm-trend | 每日告警统计（柱状图） |
| GET | /api/dashboard/active-stats | 未激活/激活占比（饼图） |
| GET | /api/dashboard/health-rank | 设备健康警示榜 TOP7 |
| GET | /api/dashboard/fault-latest | 最近故障记录 |
| GET | /api/dashboard/cycle-rank | 电池循环次数榜 TOP5 |

**GET /dashboard/summary**
```json
{ "totalDevices": 11191, "onlineDevices": 3617, "offlineDevices": 7034, "recentFaults": 4, "upgradableDevices": 50 }
```

**GET /dashboard/activity-trend**
```json
{ "dates": ["08-30","08-31"], "values": [0.95, 0.98] }
```

**GET /dashboard/alarm-trend**
```json
{ "dates": ["08-30","08-31"], "values": [50, 80] }
```

**GET /dashboard/active-stats**
```json
{ "inactive": 89.7, "active": 10.3, "total": 11191 }
```

**GET /dashboard/health-rank**
```json
[ { "imei": "864114070050327", "soh": 0 }, { "imei": "868108080100020", "soh": 70 } ]
```

**GET /dashboard/fault-latest**
```json
[ { "imei": "867584030010156", "faultType": "充电机高温故障", "level": "fault", "time": "2026-05-21 14:17:18" } ]
```

**GET /dashboard/cycle-rank**
```json
[ { "imei": "860170737687640", "cycleCount": 328 } ]
```

---

## 4. 设备管理接口

### 4.1 设备运维列表
- **GET** `/api/devices` — 分页+筛选
- 筛选参数：`imei`、`alarmType`、`faultType`、`deviceType`、`deviceStatus`（在线|离线|未激活|工作中）、`productType`（家庭储能BMS|铅改锂储能BMS|工商业储能BMS）
- 响应 data.list 项：
```json
{ "deviceNo": "111274", "imei": "0009888E00CBB34", "model": "ZLB005D1507000_00", "dealer": "兆联智控", "productType": "0", "status": "未激活", "totalVoltageMv": 80000, "assignStatus": "已绑定", "assignTime": "2026-06-09 09:36:54" }
```

### 4.2 设备分配列表
- **GET /api/devices/assign** — 参数：`imei`、`deviceNo`、`model`、`assignStatus`（未分配|已分配|已绑定）、`dealerId`、`startTime`、`endTime`
- 响应项同 4.1

### 4.3 分配设备
- **POST** `/api/devices/assign`
```json
{ "imei": "0009888E00CBB34", "dealerId": 1, "remark": "" }
```

### 4.4 解绑设备
- **PUT** `/api/devices/unbind/{imei}`

### 4.5 设备导入
- **POST** `/api/devices/import` — multipart/form-data，字段 `file`
- 响应：
```json
{ "success": 42, "failed": 0, "duplicated": 1 }
```

### 4.6 导出设备
- **GET** `/api/devices/export` — 返回二进制流（xlsx）

### 4.7 设备锁定/解锁
- **PUT** `/api/devices/{imei}/lock`
- 请求体：`{ "lockState": "locked" | "unlocked" }`

### 4.8 设备失效
- **PUT** `/api/devices/{imei}/fail`
- 请求体：`{ "reason": "硬件故障" }`

### 4.9 失效设备列表
- **GET** `/api/devices/failed` — 参数：`imei`、`failType`（硬件故障|通信中断|人为失效|长期离线）、`dealerId`、`startTime`、`endTime`
- 响应项：
```json
{ "id": 1, "deviceNo": "111201", "imei": "864114070050327", "model": "ZLB005D1507000_00", "dealer": "兆联智控", "failType": "硬件故障", "failReason": "MOS管损坏", "failTime": "2026-05-20 10:30:00", "operator": "管理员" }
```

### 4.10 恢复失效设备
- **PUT** `/api/devices/failed/{imei}/restore`

---

## 5. 设备参数接口

### 5.1 设备实时参数
- **GET** `/api/devices/{deviceNo}/params`
- 响应 data 结构：
```json
{
  "basic": { "deviceNo": "111273", "imei": "779F089B2A31D96", "batteryType": "磷酸铁锂", "batteryStrings": 16, "totalVoltage": 52.3, "current": 10.2, "soc": 85.5, "mileage": null, "soh": 92.0, "cycleCount": 328, "mosTemp": 35.2, "heatTemp": 25.0, "batteryTemp": 28.5, "protectionStatus": "正常", "systemFault": "无", "activatedAt": "2026-06-09 09:36:54", "updatedAt": "2026-09-18 16:25:30", "swVersion": "ZLB005D-V2.3.1", "sysVersion": "F0-AB3000X-VR2025042", "hwVersion": "AB3000X_H15S64Ah" },
  "switches": { "charge": true, "discharge": true, "heat": false, "preCharge": false, "currentLimit": false, "balance": true, "bluetooth": false, "parallelCharge": false, "parallelDischarge": false, "led": true, "charger": true },
  "voltage": { "maxVoltage": 3272, "minVoltage": 3263, "voltageDiff": 9 }
}
```

### 5.2 下发参数配置
- **PUT** `/api/devices/{deviceNo}/config`
- 请求体（8项阈值）：
```json
{ "cellOverVoltageThreshold": 3650, "cellUnderVoltageThreshold": 2500, "cellOverVoltageProtect": 3700, "cellUnderVoltageProtect": 2400, "voltageDiffAlarm": 100, "batteryHighTemp": 55, "batteryLowTemp": -20, "socLowAlarm": 20, "socLowProtect": 10, "dataReportInterval": 30, "heartbeatInterval": 60 }
```

### 5.3 设备操作指令
- **POST** `/api/devices/{deviceNo}/actions/{action}`
- `action` 取值：`refresh`（刷新数据）、`restart`（重启）、`gps_restart`（GPS重启）、`wake`（唤醒）、`fault_clear`（故障消除）、`sleep`（休眠）、`sync`（同步）
- 响应：`{ "message": "指令已下发" }`

### 5.4 设备历史记录
- **GET** `/api/devices/{deviceNo}/history/{type}`
- `type`：`alarm`（告警）、`fault`（故障）、`status`（状态）
- 响应：二维数组，如告警 `[["2026-05-21 14:17:18","充电机高温故障","故障","68.5℃"]]`

---

## 6. OTA接口

### 6.1 固件管理
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/ota/firmwares | 列表 |
| POST | /api/ota/firmwares | 新增 |
| PUT | /api/ota/firmwares/{id} | 修改 |
| DELETE | /api/ota/firmwares/{id} | 删除 |

固件对象：
```json
{ "id": 1, "name": "ZLB005D_V2.3.1", "version": "V2.3.1", "deviceType": "ZLB005D1507000_00", "size": "2.45", "md5": "a1b2c3d4e5f67890abcdef1234567890", "uploadTime": "2026-09-10 14:30:00", "status": "已发布", "description": "修复SOC估算偏差" }
```

### 6.2 升级任务
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/ota/tasks | 列表 |
| POST | /api/ota/tasks | 新建 |
| PUT | /api/ota/tasks/{id} | 修改 |
| DELETE | /api/ota/tasks/{id} | 删除 |
| PUT | /api/ota/tasks/{id}/start | 启动 |
| PUT | /api/ota/tasks/{id}/close | 关闭 |
| GET | /api/ota/tasks/{id} | 详情（含设备升级明细detail数组） |

任务对象：
```json
{ "id": 1, "name": "家庭储能BMS批量升级", "firmwareVersion": "V2.3.1", "totalDevices": 500, "upgraded": 487, "failed": 13, "progress": 97.4, "status": "执行中", "createdAt": "2026-09-12 08:00:00" }
```

### 6.3 升级策略
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/ota/strategies | 列表 |
| POST | /api/ota/strategies | 新建 |
| PUT | /api/ota/strategies/{id} | 修改 |
| DELETE | /api/ota/strategies/{id} | 删除 |

策略对象：
```json
{ "id": 1, "name": "夜间静默升级", "mode": "自动分组", "period": "02:00-06:00", "maxConcurrent": 50, "timeoutMinutes": 30, "status": "启用" }
```

---

## 7. 定位监控接口

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/location/device/{deviceNo} | 设备16项详情 |
| POST | /api/location/locate/{deviceNo} | 请求GPS定位 |
| GET | /api/location/devices | 设备下拉列表 |

定位响应：
```json
{ "latitude": 22.656432, "longitude": 113.923456, "online": true }
```

设备详情对象（16项）：
```json
{ "deviceNo": "111224", "imei": "ZLZK3601115030355735", "imsi": "999010000000000", "dealer": "是", "deviceType": "ZLB005D1507000_00", "productType": "1", "activated": "是", "bound": "是", "loginAllowed": "是", "hwVersion": "AB3000X_H15S64Ah", "sysVersion": "F0-AB3000X-VR2025", "swVersion": "ZLB005D-V2.3.1", "protocolVersion": "-", "createdAt": "2026-06-09 09:36:54", "updatedAt": "2026-06-15 17:14:52", "activatedAt": "2026-06-09 09:36:54", "online": true }
```

---

## 8. 商户管理接口

### 8.1 经销商管理
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/dealers | 列表（筛选：name/contact/status） |
| GET | /api/dealers/{id} | 详情 |
| POST | /api/dealers | 新增 |
| PUT | /api/dealers/{id} | 修改 |
| DELETE | /api/dealers/{id} | 删除 |

经销商对象：
```json
{ "id": 1, "code": "DL001", "name": "兆联智控", "contact": "张磊", "phone": "138****8888", "total": 8500, "online": 2800, "status": "启用", "createdAt": "2025-01-15 10:00:00", "email": "zhangl@zlzk.com", "address": "深圳市宝安区" }
```

### 8.2 经销商账户管理
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/dealer-accounts | 列表（筛选/账户名/dealerId/status） |
| POST | /api/dealer-accounts | 新增 |
| PUT | /api/dealer-accounts/{id} | 修改 |
| DELETE | /api/dealer-accounts/{id} | 删除 |
| POST | /api/dealer-accounts/{id}/reset-password | 重置密码 |

账户对象：
```json
{ "id": 1, "username": "zhang_lei", "dealer": "兆联智控", "role": "经销商管理员", "phone": "138****8888", "email": "zhangl@zlzk.com", "lastLogin": "2026-09-17 08:30:00", "status": "启用" }
```

### 8.3 角色管理
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/roles | 角色列表 |
| POST | /api/roles | 新增 |
| PUT | /api/roles/{id} | 修改 |
| DELETE | /api/roles/{id} | 删除 |
| GET | /api/roles/{id}/permissions | 权限树勾选状态 |
| PUT | /api/roles/{id}/permissions | 保存权限 `{ permissionIds: [] }` |
| GET | /api/roles/staff | 员工列表（筛选/username/role） |
| POST | /api/roles/staff | 新增员工 |
| PUT | /api/roles/staff/{id} | 修改 |
| DELETE | /api/roles/staff/{id} | 移除 |

角色对象：
```json
{ "id": 1, "name": "平台管理员", "desc": "拥有系统全部权限", "dataScope": "全部数据" }
```

员工对象：
```json
{ "id": 1, "username": "admin", "realName": "系统管理员", "role": "平台管理员", "dealer": "-", "phone": "138****8888", "status": "启用" }
```

---

## 9. 系统设置接口

### 9.1 系统配置
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/settings/system | 获取配置 |
| PUT | /api/settings/system | 保存配置 |
| PUT | /api/settings/system/reset | 恢复默认 |

配置对象：
```json
{ "systemName": "BMS管理系统", "systemVersion": "V3.2.1", "timeout": 30, "heartbeatInterval": 60, "dataRetentionDays": 180, "mapApiKey": "******", "emailNotify": "启用", "smsNotify": "启用", "alarmEmail": "alarm@bms-system.com", "alarmPhone": "13888888888" }
```

### 9.2 MQTT配置
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/settings/mqtt | 获取配置 |
| PUT | /api/settings/mqtt | 保存配置 |
| POST | /api/settings/mqtt/test | 测试连接，返回 `{ latency: 23 }` |

### 9.3 阈值配置
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/settings/threshold | 获取阈值 |
| PUT | /api/settings/threshold | 保存阈值 |
| POST | /api/settings/threshold/deploy | 批量下发，返回 `{ deployed: 3617 }` |

---

## 10. 日志接口

### 10.1 系统日志
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/logs/system | 列表（筛选/level/module/startTime/endTime） |
| DELETE | /api/logs/system | 清空日志 |

系统日志对象：
```json
{ "id": 1, "level": "INFO", "module": "设备通信", "content": "设备864114070050327心跳正常，SOC:85.5%", "imei": "864114070050327", "time": "2026-09-18 16:30:15" }
```

### 10.2 操作日志
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/logs/operation | 列表（筛选/username/startTime/endTime/result） |
| GET | /api/logs/operation/{id} | 详情 |
| GET | /api/logs/operation/export | 导出（二进制流） |

操作日志对象（11项）：
```json
{ "id": 1, "username": "管理员", "role": "平台管理员", "module": "BMS-设备管理", "content": "请求设备即时上报状态", "path": "/bms/deviceAction/syncDev", "ip": "113.87.37.53", "ipArea": "中国/广东省/深圳市/电信", "userAgent": "Chrome/152.0.0.0 Safari/537.36 Edg/152.0.0.0", "result": "失败", "time": "2026-09-12 15:26:07" }
```

### 10.3 登录锁定
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/logs/login-lock | 列表（筛选/username/ip/lockStatus） |
| PUT | /api/logs/login-lock/{id}/unlock | 手动解锁 |

锁定对象：
```json
{ "id": 1, "username": "test_01", "ip": "192.168.1.105", "failCount": 5, "lockTime": "2026-09-18 15:30:00", "unlockTime": "2026-09-18 15:45:00", "status": "已锁定" }
```

---

## 11. 枚举值定义

### 11.1 设备状态
| 值 | 含义 |
|----|------|
| online | 在线 |
| offline | 离线 |
| working | 工作中 |
| inactive | 未激活 |
| locked | 已锁定 |
| failed | 已失效 |

### 11.2 分配状态
`未分配` / `已分配` / `已绑定`

### 11.3 产品类型
`家庭储能BMS` / `铅改锂储能BMS` / `工商业储能BMS`

### 11.4 失效类型
`硬件故障` / `通信中断` / `人为失效` / `长期离线`

### 11.5 角色
`平台管理员` / `经销商管理员` / `经销商操作员` / `游客用户` / `普通测试员`

### 11.6 告警级别
`warning`（警告）/ `fault`（故障）/ `critical`（严重）

### 11.7 日志级别
`INFO` / `WARN` / `ERROR` / `DEBUG`

---

## 12. 前后端切换说明

### 12.1 当前状态
前端 `src/utils/request.js` 中：
```js
export const USE_MOCK = true
```
此时所有请求走本地 Mock 数据，页面可独立运行调试。

### 12.2 对接真实后端
1. 将 `USE_MOCK` 改为 `false`
2. 配置后端地址：`vite.config.js` 中 `proxy.target`（默认 `http://localhost:8080`）
3. 后端需实现本文件全部接口，且响应格式与 1.2 一致
4. 联调完成即可

---

*文档版本：V1.0 | 适用前端：bms-admin-frontend | 配套：BMS_MQTT接口协议文档*

> AI生成