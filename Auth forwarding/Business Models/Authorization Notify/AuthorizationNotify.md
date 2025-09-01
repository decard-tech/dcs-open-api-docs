# 授权通知（Authorisation Notification）

## 1. 概述

授权通知是DCS系统中用于实时推送用户消费授权信息的关键机制。当持卡人发起交易时，系统会向企业方发送授权通知，Enterprise需要及时响应是否同意该笔交易。本文档详细介绍了授权通知的接入方法和数据结构定义。

## 2. 安全要求

授权通知需要极高的安全等级来保证双方的资金安全，因此需要通过以下方式提高安全等级：

### 2.1 URL链接模板
```
https://{domain}/xxx/v1/auth-notification
```

### 2.2 IP白名单
请将您的服务器IP地址提供给DCS团队进行白名单配置。

### 2.3 验证签名（RSA加密）

为确保双方请求均不被篡改，DCS和Enterprise均需要生成一套RSA的公私钥，双方交换公钥。

#### 加密流程：

**DCS发送请求到Enterprise：**
1. DCS先用Enterprise's Public Key来对`authId`、`direction`、`currency`、`amount`这4个字段加密（base64编码），加密内容放在`data.encryptedData`中
2. 再把加密的内容用DCS's Private Key加签（base64编码），签名内容放在header中的`X-Auth-Signature`

**Enterprise收到请求：**
1. 从header中的`X-Auth-Signature`取出加签的信息，用DCS's Public Key来验证签名（base64编码）
2. 再拿出`data.encryptedData`的加密内容，用Enterprise's Private Key解密（base64编码），此时已经拿到了密文信息

**Enterprise返回授权处理结果给DCS：**
1. Enterprise先用DCS's Public Key来对加密前的数据结构加密（base64编码），把整个密文放在响应数据结构的`encryptData`
2. 再把加密的内容用Enterprise's Private Key加签（base64编码），签名内容放在响应数据结构的`signature`

**DCS收到授权结果：**
1. 取出响应数据结构的`signature`，用Enterprise's Public Key来验证签名（base64编码）
2. 再用DCS's Private Key对响应数据结构的`encryptData`解密（base64编码）

### 2.4 生成密钥的命令

```bash
# generate private key
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048

# generate public key
openssl rsa -pubout -in private_key.pem -out public_key.pem

# remove header and footer (-----BEGIN PUBLIC KEY---- and ----- END PUBLIC KEY-----) as well as new lines
cat public_key.pem | sed '/BEGIN/d; /END/d' | tr -d '\n'
```

### 2.5 DCS Public Key

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxhdb11OPwi0wz7bGk68E
uvPLD35jWWZJi5wc51xcUmc/i6OSTG6MXN/YKleXkYOSRIQ82h4v+oMtctH7KK9B
kcNrNosdX2dB6JYPGBynWSaaKtFTzc0tCfk3NnOqM9gjM/BwxU17RK7N3kjodqSv
At3cnsKfWZRf8gyrs0imxvM2/28NnmuU1Leil4Km5V9dpgIbeva73eMEVa4D9rx2
zziBw/iMCm5EciFc/lA9vFBjYZbVQTTgYZNFwgz2+TN+0iBGebthuV5U00le5qwm
z1t1Mv5aEaXgiDB1JdE8OAn0HkZrnAgKLH7Xwq1GVTu6k5xfTRtK0LuT7jbzUS43
bQIDAQAB
-----END PUBLIC KEY-----
```

## 3. 数据结构

### 3.1 请求数据结构

```json
{
  "webhookId": "string",
  "webhookType": "string",
  "webhookSubType": "string",
  "businessId": "string",
  "data": {
    "authId": "string",
    "direction": "string",
    "authType": "string",
    "outsId": "string",
    "originalAuthId": "string",
    "customerId": "string",
    "cardId": "string",
    "currency": "string",
    "amount": "decimal",
    "acquirerCurrency": "string",
    "acquirerAmount": "decimal",
    "cardAcceptorIdentification": "string",
    "cardAcceptorNameLocation": "string",
    "transactionType": "string",
    "encryptedData": "string"
  },
  "notificationTime": "string"
}
```

### 3.2 字段说明

#### 根级别字段

| 字段名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| webhookId | string | 是 | webhook唯一id，可以根据此字段避免webhook重复消费 |
| webhookType | string | 是 | webhook类型，固定值：AUTHORISATION |
| webhookSubType | string | 是 | webhook子类型，如：CREATE/UPDATE |
| businessId | string | 是 | webhook内容的id，此处为authId |
| data | object | 是 | 具体的webhook内容 |
| notificationTime | string | 是 | webhook通知时间，格式：yyyy-MM-dd T HH:mm:ss+08:00 |

#### 授权数据字段（data）

| 字段名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| authId | string | 是 | 授权id |
| direction | string | 是 | 方向，OUTGOING：用户付款；INCOMING：用户收款 |
| authType | string | 是 | 类型，如：NORMAL |
| outsId | string | 否 | 账单id |
| originalAuthId | string | 否 | 原始的关联授权id |
| customerId | string | 是 | 用户id |
| cardId | string | 是 | 卡id |
| currency | string | 是 | 币种 |
| amount | decimal | 是 | 金额 |
| acquirerCurrency | string | 否 | 用户请求币种 |
| acquirerAmount | decimal | 否 | 用户请求金额 |
| cardAcceptorIdentification | string | 否 | 商户号 |
| cardAcceptorNameLocation | string | 否 | 商户名称+地址 |
| transactionType | string | 是 | 交易类型：C：现金，P：还款/退货，Q：查询，R：普通消费 |
| encryptedData | string | 是 | 加密信息，加密的内容为authId、direction、currency、amount |

### 3.3 枚举值定义

#### 方向（direction）

| 值 | 描述 |
|----|------|
| OUTGOING | 用户付款，从余额账户（合作商账户）出金 |
| INCOMING | 用户收款，从余额账户（合作商账户）入金 |

#### 交易类型（transactionType）

| 值 | 描述 |
|----|------|
| C | 现金 |
| P | 还款/退货 |
| Q | 查询 |
| R | 普通消费 |

## 4. 响应数据结构

Enterprise需要返回以下格式的响应：

### 4.1 响应格式

```json
{
  "signature": "string",
  "encryptData": "string"
}
```

### 4.2 字段说明

| 字段名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| signature | string | 是 | 签名 |
| encryptData | string | 是 | 加密数据 |

### 4.3 加密前的数据结构

```json
{
  "authId": "string",
  "responseCode": "string"
}
```

| 字段名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| authId | string | 是 | 授权id |
| responseCode | string | 是 | 授权结果码 |

### 4.4 响应码定义

| 响应码 | 描述 |
|--------|------|
| 00 | 同意 |
| 01 | 拒绝,资金不足 |
| 11 | 拒绝,交易不允许 |
| 21 | 拒绝,无响应 |

## 5. 处理逻辑

### 5.1 用户付款（OUTGOING）
收到OUTGOING的Authorisation，需要冻结用户对应的资产。

### 5.2 用户收款（INCOMING）
收到INCOMING的Authorisation，需要解冻用户对应的资产。如果用户冻结的资产小于要解冻的资产，可拒绝此次Authorisation。后续会通过对账文件推送。

## 6. 示例

### 6.1 授权通知请求示例

```json
{
  "webhookId": "webhook_123456789",
  "webhookType": "AUTHORISATION",
  "webhookSubType": "CREATE",
  "businessId": "auth_987654321",
  "data": {
    "authId": "auth_987654321",
    "direction": "OUTGOING",
    "authType": "NORMAL",
    "outsId": "bill_001",
    "customerId": "user_123",
    "cardId": "card_456",
    "currency": "702",
    "amount": 100.00,
    "acquirerCurrency": "702",
    "acquirerAmount": 100.00,
    "cardAcceptorIdentification": "MERCHANT001",
    "cardAcceptorNameLocation": "TestMerchant Beijing",
    "transactionType": "R",
    "encryptedData": "encrypted_auth_data_here"
  },
  "notificationTime": "2024-01-15T10:30:00+08:00"
}
```

### 6.2 授权响应示例

```json
{
  "signature": "rsa_signature_here",
  "encryptData": "encrypted_response_data_here"
}
```

加密前的响应数据：
```json
{
  "authId": "auth_987654321",
  "responseCode": "00"
}
```

## 7. 注意事项
1. **时区**：所有时间均为UTC+8时区
2. **幂等性**：使用webhookId进行幂等性控制，避免重复处理
3. **超时时间**：请确保在合理时间内响应授权请求
4. **加密安全**：严格按照RSA加密流程处理敏感数据
5. **IP白名单**：确保服务器IP已加入白名单
6. **日志记录**：建议记录所有授权通知的处理日志

## 8. 错误处理
- 加密解密失败时，应返回相应错误码
- 签名验证失败时，拒绝处理请求
- 系统异常时，返回21（无响应）响应码
