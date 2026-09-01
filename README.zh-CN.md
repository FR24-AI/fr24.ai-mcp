# fr24.ai-mcp

[English](README.md)

Source: [FR24 International Flights MCP Integration Guide](https://fr24.ai/mcp.html)

---

## ModelScope MCP 服务配置

将 GitHub 仓库导入 ModelScope MCP 广场时，可使用以下服务配置。`X_CID` 和 `X_PASSKEY` 为用户侧环境变量，分别对应 FR24 分配的采购商 ID 和 16 位采购密钥。

```json
{
  "mcpServers": {
    "fr-flight-mcp": {
      "type": "http",
      "url": "https://mcp.fr24.ai/mcp",
      "headers": {
        "X-Cid": "${X_CID}",
        "X-Passkey": "${X_PASSKEY}"
      }
    }
  }
}
```

## 关于深圳航路

FR24 AI 是 Flightroutes24 在全球机票分销领域的资源与 AI Agent 工作流连接起来，面向 Cursor、Claude Code、Codex 等客户端提供国际机票搜索、验价、下单、订单查询等端到端能力。深圳航路成立于 2014 年，依托自研 JET-X 交易系统，连接多 GDS、航司直连、NDC 与低成本航司资源，服务全球旅行社、OTA 与航旅采购商。通过 MCP，Agent 可按“搜索航班、锁定价格、提交预订、查询订单”的标准流程调用实时票价与库存能力，使全球机票采购从人工比价和系统切换，升级为可被 AI 编排、自动化执行的智能分销链路。

---

FR24 AI MCP Server 提供国际机票搜索、验价、下单和订单查询的完整能力，支持 Cursor、Claude Code、Codex、WorkBuddy、OpenClaw 等兼容 MCP 的 AI Agent 客户端接入。

### 1. 概述

标准预订流程必须按顺序调用，不可跳步：

```text
shopping 搜索航班 -> pricing 锁定价格 -> booking 提交下单 -> orderDetail 查询订单
```

### 2. 获取 CID 与密钥

接入本服务需要 FR24 分配的采购商 ID（`X-Cid`）与 16 位采购密钥（`X-Passkey`）。

申请流程：

1. 商务对接：联系 FR24 商务团队，说明公司名称、业务场景和预估请求量。
2. 资质审核：提交企业资质材料，FR24 将在 1 个工作日内完成审核。
3. 发放凭证：审核通过后，专属商务经理将发送 `X-Cid` 与 16 位 `X-Passkey`。
4. 接入联调：按下方配置示例完成联调，可联系技术支持协助。

联系方式：

| 类型 | 信息 |
| --- | --- |
| 商务邮箱 | bd@flightroutes24.com |
| B2B 平台 | https://b2b.flightroutes24.com |
| 技术支持 | tech@flightroutes24.com |

> 密钥安全：`X-Passkey` 是核心鉴权凭证，请妥善保管，切勿提交到公共代码仓库或泄露给第三方。如密钥疑似泄露，请立即联系技术支持重置。

### 3. 接入配置

#### 3.1 服务地址

| 环境 | MCP Endpoint |
| --- | --- |
| 生产环境 | `https://mcp.fr24.ai/mcp` |

#### 3.2 鉴权方式

通过 HTTP Header 传入鉴权信息：

| Header | 说明 | 示例 |
| --- | --- | --- |
| `X-Cid` | 采购商 ID，由 FR24 分配 | `FRG` |
| `X-Passkey` | 采购密钥，必须是 16 位字符串 | `your16charpasskey` |

`X-Passkey` 必须恰好 16 个字节，长度不符将返回鉴权失败错误。

#### 3.3 客户端配置示例

所有客户端使用相同鉴权信息：`X-Cid` 为采购商 ID，`X-Passkey` 为 16 位采购密钥。请将示例中的 `YOUR_CID` 和 `YOUR_16CHAR_PASSKEY` 替换为 FR24 分配的真实凭证。

Cursor（项目根目录 `.cursor/mcp.json`）：

```json
{
  "mcpServers": {
    "fr-flight-mcp": {
      "url": "https://mcp.fr24.ai/mcp",
      "headers": {
        "X-Cid": "YOUR_CID",
        "X-Passkey": "YOUR_16CHAR_PASSKEY"
      }
    }
  }
}
```

Claude Code（项目级 `.mcp.json` 或用户级 `~/.mcp.json`）：

```json
{
  "mcpServers": {
    "fr-flight-mcp": {
      "type": "http",
      "url": "https://mcp.fr24.ai/mcp",
      "headers": {
        "X-Cid": "YOUR_CID",
        "X-Passkey": "YOUR_16CHAR_PASSKEY"
      }
    }
  }
}
```

也可以通过命令行添加：

```bash
claude mcp add --transport http fr-flight-mcp https://mcp.fr24.ai/mcp \
  -H "X-Cid:YOUR_CID" \
  -H "X-Passkey:YOUR_16CHAR_PASSKEY"
```

Codex（`~/.codex/config.toml`）：

```toml
[mcp_servers.fr-flight-mcp]
transport = "http"
url = "https://mcp.fr24.ai/mcp"

[mcp_servers.fr-flight-mcp.headers]
X-Cid = "YOUR_CID"
X-Passkey = "YOUR_16CHAR_PASSKEY"
```

WorkBuddy（设置中心新增 HTTP 类型 Server，或编辑 `workbuddy.mcp.json`）：

```json
{
  "servers": {
    "fr-flight-mcp": {
      "transport": "http",
      "endpoint": "https://mcp.fr24.ai/mcp",
      "headers": {
        "X-Cid": "YOUR_CID",
        "X-Passkey": "YOUR_16CHAR_PASSKEY"
      }
    }
  }
}
```

OpenClaw（`openclaw.config.json` 或 `~/.openclaw/mcp_servers.json`）：

```json
{
  "mcpServers": {
    "fr-flight-mcp": {
      "transport": "streamable-http",
      "url": "https://mcp.fr24.ai/mcp",
      "headers": {
        "X-Cid": "YOUR_CID",
        "X-Passkey": "YOUR_16CHAR_PASSKEY"
      }
    }
  }
}
```

### 4. 工具详解

#### shopping

搜索国际机票，支持 6 种搜索模式，最多返回 10 条航班。

必填参数：

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `origin` | string | 出发地 IATA 三字码，如 `PEK`、`HKG` |
| `destination` | string | 目的地 IATA 三字码，如 `BKK`、`NRT` |
| `depDate` | string | 出发日期，格式 `yyyy-MM-dd` |

可选参数：

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `adultNum` | integer | 成人人数，默认 1 |
| `childNum` | integer | 儿童人数（2-12 岁），默认 0 |
| `infantNum` | integer | 婴儿人数（不足 2 岁），默认 0 |
| `retDate` | string | 返程日期，单程不传 |
| `searchMode` | string | 搜索模式，见下表 |
| `departureTimeRange` | string | `TIME` 模式参数，如 `06:00-12:00` |
| `priceRange` | string | `PRICE` 模式参数，如 `500-2000` |
| `flightNo` | string | `FLIGHT` 模式参数，如 `CA1234` |
| `cabin` | string | `CABIN` 模式参数，`Y`=经济，`C`=商务，`F`=头等，`P`=超经 |

搜索模式：

| 值 | 说明 | 配合参数 |
| --- | --- | --- |
| 不传 | 默认搜索，按价格排序 | - |
| `TIME` | 按出发时间段过滤 | `departureTimeRange` |
| `PRICE` | 按总价区间过滤 | `priceRange` |
| `FLIGHT` | 指定航班号搜索 | `flightNo` |
| `TRANSFER` | 只返回中转航班 | - |
| `CABIN` | 按舱位过滤 | `cabin` |

调用示例：

```json
{ "origin": "HKG", "destination": "BKK", "depDate": "2026-08-01" }
```

```json
{
  "origin": "PEK",
  "destination": "NRT",
  "depDate": "2026-08-10",
  "searchMode": "TIME",
  "departureTimeRange": "06:00-12:00"
}
```

```json
{
  "origin": "SZX",
  "destination": "BKK",
  "depDate": "2026-08-15",
  "retDate": "2026-08-20"
}
```

```json
{
  "origin": "HKG",
  "destination": "SIN",
  "depDate": "2026-08-01",
  "adultNum": 1,
  "childNum": 1
}
```

返回字段：

| 字段 | 说明 |
| --- | --- |
| `flightNo` | 航班号 |
| `airline` | 航司名称 |
| `cabin` | 舱位（经济舱 / 商务舱 / 头等舱） |
| `depAirport` | 出发机场，格式 `机场码(城市名)` |
| `depTime` | 出发时间，格式 `MM-dd HH:mm` |
| `arrAirport` | 到达机场 |
| `arrTime` | 到达时间 |
| `duration` | 飞行时长，如 `3h45m` |
| `type` | 直飞 / 中转 |
| `priceDisplay` | 总价含币种，如 `1200 CNY` |
| `remainingSeats` | 剩余座位数 |
| `seatsAlert` | 剩余小于等于 3 座时的紧张提示 |
| `refundPolicy` | 退改签政策 |

#### pricing

锁定选定航班的价格，有效期 30 分钟。必须在 `shopping` 之后、`booking` 之前调用。带儿童或婴儿时，`adultNum`、`childNum`、`infantNum` 必须与 `shopping` 时传入的人数一致。

必填参数：

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `offerId` | string | `shopping` 返回的 `offerId` |

可选参数：

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `adultNum` | integer | 成人人数，默认 1，需与搜索时一致 |
| `childNum` | integer | 儿童人数，默认 0 |
| `infantNum` | integer | 婴儿人数，默认 0 |

返回字段：

| 字段 | 位置 | 说明 |
| --- | --- | --- |
| `priceDisplay` | 顶层 | 锁定总价，含币种 |
| `priceBreakdown` | 顶层 | 分项价格明细（多人时展示） |
| `remainingSeats` | 顶层 | 剩余座位 |
| `msg` | 顶层 | 提示信息 |
| `meta.verifyOfferId` | `meta` | 用于 `booking` 的 `verifyOfferId` |
| `meta.isInternational` | `meta` | 是否国际航线 |
| `meta.requiredPassengerFields` | `meta` | 乘客必填字段，`cardType.allowedValues` 为允许的证件类型 |

`requiredPassengerFields` 示例：

```json
{
  "birthday": true,
  "gender": true,
  "cardNum": true,
  "cardType": {
    "required": true,
    "allowedValues": ["PP", "GA", "HX", "TB"]
  },
  "cardIssuedPlace": true,
  "cardExpiryDate": true,
  "nationality": true,
  "paxEmail": false,
  "paxMobile": false
}
```

国际航线的 `allowedValues` 不含 `ID`（身份证），境内航线可包含 `ID`。下单时 `cardType` 必须从该列表中选择。

#### booking

提交机票预订，支持多人（成人、儿童、婴儿）。

必填参数：

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `verifyOfferId` | string | `pricing` 返回的 `meta.verifyOfferId` |
| `passengersJson` | string | 乘客列表 JSON 数组字符串 |

可选参数：

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `allowedCardTypes` | array | `pricing` 返回的 `cardType.allowedValues`，强烈建议传入 |
| `departureDate` | string | 出发日期，用于年龄校验，可从缓存自动获取 |
| `contactName` | string | 联系人姓名，不填则用第一个乘客兜底 |
| `contactMobile` | string | 联系人手机号 |
| `contactEmail` | string | 联系人邮箱 |
| `idempotencyKey` | string | 幂等键（UUID），24 小时内相同 key 只下一次单 |
| `partnerOrderNo` | string | 合作方自定义订单号 |

`passengersJson` 乘客字段：

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `name` | 必填 | 姓/名，英文大写，如 `ZHANG/SAN` |
| `paxType` | 必填 | `ADT`=成人，`CHD`=儿童，`INF`=婴儿 |
| `gender` | 必填 | `M`=男，`F`=女 |
| `birthday` | 必填 | 出生日期，格式 `yyyy-MM-dd` |
| `cardType` | 必填 | 证件类型，必须在 `allowedValues` 中选 |
| `cardNum` | 必填 | 证件号码，最长 20 位 |
| `cardIssuedPlace` | 按需 | 证件签发地，ISO 两字码，如 `CN` |
| `cardExpiryDate` | 按需 | 证件有效期，格式 `yyyy-MM-dd` |
| `nationality` | 按需 | 国籍，ISO 两字码，如 `CN` |
| `areaCode` | 可选 | 手机区号，默认 `86` |
| `paxMobile` | 可选 | 联系电话 |
| `paxEmail` | 可选 | 联系邮箱 |
| `accompaniedPaxId` | CHD/INF 必填 | 关联成人序号（从 1 起），如 `"1"` |

证件类型：

| 代码 | 说明 | 适用航线 |
| --- | --- | --- |
| `ID` | 身份证 | 仅境内航线 |
| `PP` | 护照 | 国际 / 境内均可 |
| `GA` | 港澳通行证 | 国际 / 境内均可 |
| `HX` | 回乡证 | 国际 / 境内均可 |
| `TB` | 台胞证 | 国际 / 境内均可 |

年龄规则：

| `paxType` | 出发时年龄 | 占座 |
| --- | --- | --- |
| `INF` | 不足 2 岁 | 不占座 |
| `CHD` | 2 岁（含）至 12 岁（不含） | 占座 |
| `ADT` | 12 岁及以上 | 占座 |

国际航线禁止使用身份证（`ID`）。每名成人最多携带 2 名儿童和 1 名婴儿。

#### orderDetail

查询已下单的机票订单详情。

必填参数：

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `orderNo` | string | `booking` 返回的订单号 |

返回字段：

| 字段 | 说明 |
| --- | --- |
| `orderNo` | 订单号 |
| `orderStatus` | 订单状态 |
| `priceDisplay` | 总价 |
| `createTime` | 下单时间 |
| `segments` | 航班航段信息 |
| `passengers` | 乘客信息 |
| `ticketNos` | 出票票号（出票后才有） |

### 5. 完整调用示例

以下为单成人预订香港到曼谷国际机票的完整四步流程：

1. 搜索航班

```json
{ "origin": "HKG", "destination": "BKK", "depDate": "2026-08-01" }
```

从返回的 `flights` 列表中选择 `offerId` 传给下一步。

2. 验价锁定

```json
{ "offerId": "21725576870629376120", "adultNum": 1 }
```

从返回的 `meta` 中取 `verifyOfferId` 和 `requiredPassengerFields.cardType.allowedValues`。

3. 提交下单

```json
{
  "verifyOfferId": "2172558470381977600",
  "passengersJson": "[{\"name\":\"ZHANG/SAN\",\"paxType\":\"ADT\",\"gender\":\"M\",\"birthday\":\"1990-01-01\",\"cardType\":\"PP\",\"cardNum\":\"E12345678\",\"cardIssuedPlace\":\"CN\",\"cardExpiryDate\":\"2030-12-31\",\"nationality\":\"CN\"}]",
  "allowedCardTypes": ["PP", "GA", "HX", "TB"]
}
```

返回 `orderNo` 和 `payUrl` 后，引导用户前往支付。

4. 查询订单

```json
{ "orderNo": "21725493286277120" }
```

### 6. 常见错误码

| 错误码 | 说明 | 处理建议 |
| --- | --- | --- |
| `AUTH_FAILED` | 鉴权失败 | 检查 `X-Cid` 和 `X-Passkey`，passkey 必须恰好 16 位 |
| `PARAM_INVALID` | 参数校验不通过 | 查看 `msg` 字段了解具体原因，如姓名格式、证件类型或年龄不符 |
| `10701298` | `offerId` 已失效（之前验价失败） | 重新调用 `shopping` 获取新的 `offerId` |
| `20901997` | 验价失败 | 人数与搜索时不一致，或该航班不支持儿童票 |
| `OFFER_ID_EXPIRED` | `offerId` 已过期（超过 30 分钟） | 重新搜索获取新 `offerId` |
| `PASSENGER_NUM_DIFF` | 乘客人数与验价不一致 | 确保 `booking` 的乘客数量与 `pricing` 时一致 |
| `ACCOMPANIED_PAX_ID` | `accompaniedPaxId` 无效 | CHD/INF 的 `accompaniedPaxId` 必须指向列表中存在的 ADT |

### 7. 注意事项

1. 流程顺序不可跳过：必须按 `shopping -> pricing -> booking` 顺序调用，不能直接 `booking`。
2. 验价有效期：`pricing` 锁价后 30 分钟内有效，超时需重新验价。
3. 儿童票搜索：搜索时即需传入 `childNum`，验价时人数必须与搜索时一致。
4. 国际航线证件：国际航线不支持身份证（`ID`），必须使用护照（`PP`）等国际证件。
5. 婴儿规则：`INF` 不占座；每名成人只能携带 1 名婴儿，最多携带 2 名儿童。
6. 幂等下单：网络不稳定场景建议传入 `idempotencyKey`（UUID 格式），防止重复下单。
7. 支付入口：下单成功后通过 `payUrl` 跳转至 B2B 订单页面完成支付。

---
