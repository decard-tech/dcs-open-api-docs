# Webhook 接入指南

## 概述

Webhook是用于向企业实时推送业务状态变更的机制。当卡订单状态发生变化、卡状态更新或授权结果产生时，系统会主动向企业配置的webhook地址发送通知。

## 接入准备

### 1. 配置Webhook URL
在Enterprise配置中需要提供以下信息：
- **webhookUrl**: webhook通知地址
- **IP白名单**: 提供允许接收请求的IP地址给我们

### 2. 安全验证

#### IP白名单
请将您的服务器IP地址提供给我们进行白名单配置。

#### 签名验证
Webhook请求的header中包含`X-Signature`属性，使用HmacSHA256算法验证签名。

**验证示例代码：**
```java
import org.apache.commons.codec.binary.Hex;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

class HmacSignature {
    public static void main(String[] args) throws NoSuchAlgorithmException, InvalidKeyException {
        String secretKey = "secret_key";
        String webhookPayloadStr = "webhook_Payload_Str";
        byte[] hmacSha256;
        SecretKeySpec secretKeySpec = new SecretKeySpec(secretKey.getBytes(), "HmacSHA256");
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(secretKeySpec);
        hmacSha256 = mac.doFinal(webhookPayloadStr.getBytes());
        System.out.println(Hex.encodeHexString(hmacSha256));
    }
}
```

## 公共数据结构

所有webhook请求都包含以下公共字段：

| 字段 | 类型 | 描述 |
|------|------|------|
| webhookId | string | webhook唯一id，可根据此字段避免webhook重复消费 |
| webhookType | string | webhook类型，如：CARD_ORDER/CARD/AUTHORISATION_RESULT |
| webhookSubType | string | webhook子类型，如：CREATE/UPDATE |
| businessId | string | webhook内容的id，如：cardOrderId/cardId/authId |
| data | object | 具体的webhook内容 |
| notificationTime | string | webhook通知时间，格式：yyyy-MM-dd T HH:mm:ss+08:00 |

## 具体Webhook类型

### 1. Card Order Webhook

**触发场景**: 卡订单状态发生变化时

**webhookType**: `CARD_ORDER`

**data字段结构**:

| 字段 | 类型 | 描述 |
|------|------|------|
| cardOrderId | string | 卡订单ID |
| profileId | string | 卡配置id |
| type | string | 卡类型：VIRTUAL/VIRTUAL_TO_PHYSICAL/REPLACEMENT |
| customerId | string | 用户id |
| cardId | string | 卡id，当且仅当status=SUCCEED时才有值 |
| status | string | 卡订单状态 |
| errorCode | string | 错误码 |
| errorReason | string | 错误原因 |
| cardOrderRef | string | 卡订单幂等字段 |
| needEddFile | boolean | 需要额外上传EDD：true需要/false不需要 |
| replaceCardId | string | 替换的cardId（仅VIRTUAL_TO_PHYSICAL/REPLACEMENT类型使用） |
| createTime | string | 创建时间 |
| modifyTime | string | 最后更新时间 |

**状态说明**:
- `type=VIRTUAL`: PENDING/CUSTOMER_PASS/KYC_PASS/CHANNEL_CUSTOMER_PASS/SUCCEED/FAILED
- `type=VIRTUAL_TO_PHYSICAL`: PENDING/PHYSICAL_SETTING_COMPLETED/SUCCEED/FAILED
- `type=REPLACEMENT`: PENDING/SUCCEED/FAILED

### 2. Card Webhook

**触发场景**: 卡状态发生变化时

**webhookType**: `CARD`

**data字段结构**:

| 字段 | 类型 | 描述 |
|------|------|------|
| cardId | string | 卡id |
| enterpriseId | string | 企业id |
| profileId | string | 卡配置id |
| type | string | 卡类型：PHYSICAL/VIRTUAL |
| customerId | string | 用户Id |
| status | string | 卡状态：WAITING_ACTIVE/ACTIVATED/FROZEN/BLOCKED/INVALID |
| errorCode | string | 错误码 |
| errorReason | string | 错误原因 |
| panFirst6 | string | 卡号前6位 |
| panLast4 | string | 卡号后4位 |
| createTime | string | 创建时间 |
| modifyTime | string | 最后更新时间 |

### 3. Authorisation Result Webhook

**触发场景**: 授权处理完成时

**webhookType**: `AUTHORISATION_RESULT`

**data字段结构**:

| 字段 | 类型 | 描述 |
|------|------|------|
| authId | string | 授权id |
| approveFlag | string | 授权状态：A(approve)/D(decline) |
| rejectReason | string | 授权拒绝原因 |
| approveDate | string | 授权同意时间，格式：yyyy-MM-dd T HH:mm:ss+08:00 |

### 4. Authorisation 3ds Challenge Webhook

**触发场景**: 3DS挑战状态变更时

**webhookType**: `AUTHORISATION_3DS_CHALLENGE`

**data字段结构**:

| 字段 | 类型 | 描述 |
|------|------|------|
| challengeId | string | 发起交易前的3ds挑战id |
| status | string | 状态：INIT/NOTICED/RECEIVED/APPROVED/REJECTED |
| cardId | string | 卡id |
| expiryTime | string | 挑战过期时间 |
| currency | string | 发起交易的币种 |
| amount | number | 发起交易的金额 |
| merchantId | string | 商户id |
| merchantName | string | 商户名称 |
| merchantCountry | string | 商户国家 |
| mcc | string | 商户类别 |

### 5. Intent Ticket Webhook

**触发场景**: 工单状态变更时

**webhookType**: `INTENT_TICKET`

**data字段结构**:

| 字段 | 类型 | 描述 |
|------|------|------|
| ticketId | string | 工单id |
| ticketType | string | 工单类型：CREATE_CARD_EDD/INCREASE_LIMIT_EDD |
| ticketStatus | string | 工单状态：INIT/SUBMIT_COMPETED/CHECK_PASS/SUCCEED/FAILED |
| ticketRef | string | 外部唯一id |
| createTime | string | 创建时间 |
| modifyTime | string | 最后更新时间 |

## 接入最佳实践

### 1. 幂等性处理
使用`webhookId`字段确保同一webhook消息不会被重复处理。

### 2. 响应要求
- Webhook接收端应返回HTTP 200状态码表示处理成功
- 处理失败时返回非200状态码，系统会进行重试（重试3次）

### 3. 安全建议
- 验证请求来源IP是否在白名单中
- 验证X-Signature签名确保请求未被篡改
- 使用HTTPS确保传输安全


## 注意事项

1. 所有时间字段均为UTC+8时区
2. 币种字段使用ISO货币代码3位数字代码
3. 国家码使用ISO国家码2位字母代码
4. Webhook通知具有时效性，建议及时处理避免超时

## 技术支持

如需技术支持或有疑问，请联系我们获取帮助。