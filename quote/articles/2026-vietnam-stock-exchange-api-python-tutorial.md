---
title: '2026 越南证券交易所（VN30, HOSE）API 接口指南'
description: '2026年最新越南股票API接入教程，包含VN30指数和HOSE交易所实时数据获取，提供完整Python代码示例，涵盖实时行情、历史数据和财务分析'
coverImage: 'canada-stock-exchange.webp'
keywords: '越南股票API, VN30 API, HOSE API, 2026, Python股票数据, 越南证券交易所, 实时行情API'
publishDate: '2026-01-13'
tags: ['越南股票API','VN30 API','越南证券交易所', '实时行情API']
---

越南证券交易所（主要是胡志明市证券交易所 HOSE，其核心指数为 VN30）近年来备受关注。作为新兴市场的重要组成部分，HOSE 及其 VN30 成分股吸引了众多投资者的兴趣。为获取稳定的实时行情、历史数据和盘口信息，选择合适的 API 提供商至关重要。本指南概述主流 API 服务商的比较，并提供 Python 对接示例，以协助开发者高效接入相关数据。

## 一、核心概述

- **越南证券交易所核心组成**：主要包括 HOSE（胡志明市）和 HNX（河内）
- **实时行情 API**：用于获取最新报价、成交量等动态数据
- **历史数据 API**：用于回溯 K 线（分钟/日/周线等）
- **盘口数据**：包含买一到买五、卖一到卖五的价格和挂单量，适用于高频交易场景

## 二、主流越南 HOSE（VN30）API 提供商对比

目前支持越南 HOSE 市场的 API 提供商包括以下三个主要选项，分别从数据覆盖、稳定性、成本等核心维度进行对比：

### 1. iTick API

**核心优势**：

- 全球多市场覆盖，对越南 HOSE 支持完善
- 支持 VN30 成分股的实时报价、历史行情、Level2 盘口数据
- 支持 RESTful 和 WebSocket 双协议
- 免费套餐足够日常测试，付费版性价比高

**技术规格**：

- 延迟低至 100ms 内，满足非超高频交易需求
- 历史数据覆盖 30 年以上，支持分钟级到日线级 K 线
- 注册即送 API Token，无需信用卡
- Python 有成熟客户端库，对接成本低

### 2. RPDS DATA

**核心优势**：

- 覆盖全球多个新兴市场，包括越南 HOSE、印度、马来西亚等
- 数据清洗能力强，异常率低
- 支持 HTTP 和 WS 双协议，全量获取市场数据

**技术规格**：

- 稳定性良好
- 适合学术研究或专业量化团队，按数据量付费

**限制**：针对越南市场的定制化文档较少，新手对接需多花时间调试

### 3. Bloomberg API

**核心优势**：

- 全球金融数据领域标杆，深度覆盖越南 HOSE 市场及 VN30 成分股
- 数据包含实时行情、深度盘口、历史 K 线及基本面数据
- 系统可用性达 99.99%，依托全球分布式架构保障稳定
- 自带完善的合规风控模块，适配机构监管需求

**技术规格**：

- 支持 VN30 成分股全量数据，包括 Level2 盘口及 Tick 级历史数据
- 延迟低至毫秒级

**限制**：对个人开发者和中小团队极不友好，无独立免费套餐，终端及 API 服务年费高昂，且学习门槛较高

## 三、Python 对接 HOSE（VN30）技术实现

下面通过 iTick API ，展示如何通过 Python 实现与越南 HOSE 的对接。

### 环境配置

#### 获取 API Token

- 访问 [iTick 官网](https://itick.org/zh-cn)，完成注册
- 在个人中心找到 API Token，此为调用接口的核心凭证

#### 安装依赖库

需要安装 requests（调用 REST 接口）和 websocket-client（订阅实时行情）：

```bash
pip install requests websocket-client
```

### 核心接口调用示例

以下示例均针对 HOSE 的 VN30 成分股。

#### REST API 获取 VN30 成分股实时报价

此方法适合单次获取最新价格、涨跌幅、成交量等基础数据：

```python
import requests

# 替换为您的iTick API Token
ITICK_API_TOKEN = "YOUR_API_TOKEN"
# 目标标的：越南HOSE市场VN30成分股——越南航空（HVN）
symbol = "HVN"
# 实时报价API接口地址
url = f"https://api.itick.org/stock/quote?region=VN&code={symbol}"

# 请求头（必须包含token）
headers = {
    "accept": "application/json",
    "token": ITICK_API_TOKEN
}

# 发送请求并处理响应
response = requests.get(url, headers=headers)
if response.status_code == 200:
    data = response.json()
    # 解析核心数据（最新价、开盘价、涨跌幅、成交量）
    stock_data = data.get("data", {})
    print(f"标的：{symbol}")
    print(f"最新价：{stock_data.get('ld')} 越南盾")
    print(f"开盘价：{stock_data.get('o')} 越南盾")
    print(f"涨跌幅：{stock_data.get('chp')}%")
    print(f"当前成交量：{stock_data.get('v')} 股")
else:
    print(f"请求失败，状态码：{response.status_code}，错误信息：{response.json().get('message')}")
```

#### 获取 VN30 成分股历史行情（K 线数据）

此方法适合策略回测，支持分钟级、日线级等多种周期（其他周期请参考 [iTick 官方文档](https://docs.itick.org/zh-cn)）：

```python
import requests

ITICK_API_TOKEN = "YOUR_API_TOKEN"
symbol = "HVN"  # 越南航空
# 历史K线API接口（kType=8代表日线，limit=100代表获取100根K线）
url = f"https://api.itick.org/stock/kline?region=VN&code={symbol}&kType=8&limit=100"

headers = {
    "accept": "application/json",
    "token": ITICK_API_TOKEN
}

response = requests.get(url, headers=headers)
if response.status_code == 200:
    data = response.json()
    kline_list = data.get("data", [])
    print(f"获取{symbol}最近100根日线数据：")
    for kline in kline_list:
        # 解析K线数据（时间戳、开高低收、成交量）
        print(f"时间：{kline.get('t')}，开盘：{kline.get('o')}，最高：{kline.get('h')}，最低：{kline.get('l')}，收盘：{kline.get('c')}，成交量：{kline.get('v')}")
else:
    print(f"请求失败，状态码：{response.status_code}，错误信息：{response.json().get('message')}")
```

#### WebSocket 订阅 VN30 成分股实时盘口数据

此方法适合需要持续监控盘口变化的场景，可实时获取买一到买五、卖一到卖五的价格和挂单量：

```python
import websocket
import json

ITICK_API_TOKEN = "YOUR_API_TOKEN"
symbol = "VN$HVN"  # 越南航空

def on_open(ws):
    # 连接成功后，发送订阅请求
    subscribe_msg = {
        "ac": "subscribe",
        "params": symbol,
        "types": "depth" # 盘口数据
    }
    ws.send(json.dumps(subscribe_msg))
    print("WebSocket连接成功，已订阅盘口数据...")

def on_message(ws, message):
    # 接收并解析实时盘口数据
    data = json.loads(message)
    if data.get("type") == "depth":
        depth_data = data.get("data", {})
        buy_depth = depth_data.get("b", [])  # 买盘数据（买一到买五）
        sell_depth = depth_data.get("a", [])  # 卖盘数据（卖一到卖五）
        print(f"\n{symbol} 实时盘口（{depth_data.get('t')}）：")
        print("卖盘：", [(f"卖{i+1}", f"{item.get('p')}盾", f"{item.get('v')}股") for i, item in enumerate(sell_depth[:5])])
        print("买盘：", [(f"买{i+1}", f"{item.get('p')}盾", f"{item.get('v')}股") for i, item in enumerate(buy_depth[:5])])

def on_error(ws, error):
    print(f"连接错误：{error}")

def on_close(ws, close_status_code, close_msg):
    print(f"连接关闭，状态码：{close_status_code}，信息：{close_msg}")

# 建立WebSocket连接
ws_url = "wss://api.itick.org/stock"
ws = websocket.WebSocketApp(ws_url,
                            on_open=on_open,
                            on_message=on_message,
                            on_error=on_error,
                            on_close=on_close)
# 持续运行连接
ws.run_forever()
```

## 四、重要提醒

1. **订阅限制**：免费计划有订阅上限；如果需要很多个标的，建议考虑升级套餐
2. **安全要求**：请勿将 API Token 泄露给他人，一旦泄露，将无法访问 API 接口
3. **频率控制**：免费计划请勿频繁调用 API 接口，否则容易触发 API 频率限制

## 五、总结

对于计划进入越南股市量化领域的开发者而言，iTick API 是一个理想的解决方案。它不仅提供了全面的越南证券交易所（HOSE）和 VN30 指数数据，还具备以下显著优势：

- **全面的数据覆盖**：支持实时报价、历史行情和 Level2 盘口数据，满足各种量化分析需求
- **卓越的性能表现**：延迟低至 100ms 内，历史数据覆盖 30 年以上
- **灵活的接入方式**：支持 RESTful 和 WebSocket 双协议，适应不同应用场景
- **友好的定价策略**：免费套餐适合日常测试，付费版本性价比高，适合商业应用

通过本文提供的 Python 示例代码，开发者可以快速集成 iTick API，轻松获取越南证券市场的实时和历史数据，为量化策略开发和数据分析奠定坚实基础。随着越南资本市场的不断发展，iTick API 将继续为开发者提供稳定可靠的数据服务，助力量化交易的成功实施。

官方文档：[https://docs.itick.org](https://docs.itick.org/zh-cn)\
GitHub：[https://github.com/itick-org](https://github.com/itick-org)