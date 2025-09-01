# 环境配置

## 环境概述

DCS Standard Authorization 模式提供两套环境供开发者使用：

| 环境   | 用途   | API 域名                    | WebSocket 域名               |
|------|------|---------------------------|----------------------------|
| 测试环境 | 开发测试 | https://api.uatdcd.com    | wss://stream.uatdcd.com    |
| 生产环境 | 正式服务 | https://api.thedecard.com | wss://stream.thedecard.com |

## 测试环境

### 特点
- 用于开发和测试
- 数据不会影响生产环境
- 支持所有 API 功能
- 可以使用测试数据


### 测试数据
- 测试环境支持模拟 KYC 流程
- 可以使用测试手机号进行注册
- 支持模拟卡片申请和交易

## 生产环境

### 特点
- 正式商业环境
- 真实资金和交易
- 严格的安全验证
- 完整的合规流程

### 安全要求
- 必须使用生产环境 AK/SK
- 需要通过 IP 白名单验证
- 所有操作都会产生真实影响

## AKSK 配置

### 测试环境 AKSK
- 通过商务团队直接获取
- 用于开发和测试阶段
- 提供测试环境IP

### 生产环境 AKSK
生产环境的 AKSK 需要通过安全流程获取：

1. **提供信息**
   - 安全邮箱地址
   - 获取 AK/SK 的服务器 IP

2. **接收邮件**
   - 邮箱会收到临时安全链接
   - 链接仅一次有效

3. **提取密钥**
   - 使用 extractUrl + extractSecretKey
   - 在指定 IP 上执行提取
   
详情请参考：[生产环境的AKSK安全传输](../../API%20Reference/flow/readme.md#生产环境的aksk安全传输)
## 网络配置

### 请求头设置
```http
Content-Type: application/json
API-KEY: your_api_key
TIMESTAMP: 1693123456789
NONCE: random_string
SIGNATURE: generated_signature
```

## 错误处理

### 网络错误
- 连接失败：检查网络和域名
- 超时：增加超时时间或重试
- 证书错误：确认 HTTPS 配置

### 认证错误
- 签名错误：检查签名算法和参数
- 时间戳过期：确保时间同步
- IP 限制：联系技术支持添加白名单
