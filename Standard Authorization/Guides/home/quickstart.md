# 快速开始指南

本指南将帮助您快速完成 DCS Standard Authorization 模式的基本集成。

## 前置条件

1. 获得 API Key 和 Secret Key
2. 确认 IP 已加入白名单
3. 准备开发环境

## 步骤 1：环境配置

### 选择环境
```properties
## 测试环境
BASE_URL = https://api.uatdcd.com

## 生产环境
## BASE_URL = https://api.thedecard.com
```

### 配置认证信息
```properties
  apiKey=your_api_key
  secretKey=your_secret_key
  baseURL=BASE_URL
```

## 步骤 2：实现签名算法

```javascript
const crypto = require('crypto');

function generateSignature(apiKey, timestamp, nonce, payload, secretKey) {
  const message = apiKey + timestamp + nonce + payload;
  return crypto
    .createHmac('sha256', secretKey)
    .update(message)
    .digest('hex');
}

function createHeaders(method, url, body = '') {
  const timestamp = Date.now().toString();
  const nonce = Math.random().toString(36).substring(2);
  
  // 构造 payload
  let payload = '';
  if (method === 'GET') {
    const urlObj = new URL(url);
    payload = urlObj.search.substring(1); // 去掉 '?'
  } else {
    payload = body;
  }
  
  const signature = generateSignature(
    config.apiKey,
    timestamp,
    nonce,
    payload,
    config.secretKey
  );
  
  return {
    'Content-Type': 'application/json',
    'API-KEY': config.apiKey,
    'TIMESTAMP': timestamp,
    'NONCE': nonce,
    'SIGNATURE': signature
  };
}
```

## 步骤 3：发送验证码

```
curl --location 'https://api.uatdcd.com/captcha/v1/send-mobile-code' \
--header 'X-DAPI-API-KEY: 2DF34A4C49984142A37B0BA7843779BB' \
--header 'X-DAPI-SIGN: cdce008d72f75f0059cb5df8909fb0edb4fb4c65e4c1ec22a6b6a6b1e6648249' \
--header 'X-DAPI-TIMESTAMP: 1756707558921' \
--header 'X-DAPI-NONCE: 37620' \
--header 'Content-Type: application/json' \
--data '{
"mobileCode":"CN",
"mobile": "17600113032",
"behavioral":"REGISTER"
}'
```

## 步骤 4：用户注册

```
curl --location 'https://api.uatdcd.com/account/v1/register' \
--header 'X-DAPI-API-KEY: 2DF34A4C49984142A37B0BA7843779BB' \
--header 'X-DAPI-SIGN: cdce008d72f75f0059cb5df8909fb0edb4fb4c65e4c1ec22a6b6a6b1e6648249' \
--header 'X-DAPI-TIMESTAMP: 1756707558921' \
--header 'X-DAPI-NONCE: 37620' \
--header 'x-gray-env: openapi' \
--header 'Content-Type: application/json' \
--data '{
    "mobileCode":"CN",
    "mobile": "17600113032",
    "smsCode":"677376"
}'
```

## 步骤 5：获取引导页链接

```
curl --location 'https://api.uatdcd.com/redirect/v1/guidance-link' \
--header 'X-DAPI-API-KEY: 2DF34A4C49984142A37B0BA7843779BB' \
--header 'X-DAPI-SIGN: cdce008d72f75f0059cb5df8909fb0edb4fb4c65e4c1ec22a6b6a6b1e6648249' \
--header 'X-DAPI-TIMESTAMP: 1756707558921' \
--header 'X-DAPI-NONCE: 37620' \
--header 'Content-Type: application/json' \
--data '{
    "action":"KYC_GUIDE", 
    "externalUserId": "be2e6668-8107-42b5-ae16-08f6ca6eb48d",
    "successRedirectUrl":"https://www.baidu.com",
    "errorRedirectUrl":"https://www.google.com",
    "cardMantissa":"1111",
    "language":"en",
    "userAgent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36"
}'
```

## 步骤 6：查询用户资产

```
curl --location 'https://api.uatdcd.com/user-asset/v1/balance?externalUserId=46438c1a-3bec-464d-93ca-c69699b54e8e' \
--header 'X-DAPI-API-KEY: 2DF34A4C49984142A37B0BA7843779BB' \
--header 'X-DAPI-SIGN: cdce008d72f75f0059cb5df8909fb0edb4fb4c65e4c1ec22a6b6a6b1e6648249' \
--header 'X-DAPI-TIMESTAMP: 1756707558921' \
--header 'X-DAPI-NONCE: 37620'
```

## 完整示例伪代码

```javascript
async function quickStart() {
  try {
    // 1. 发送验证码
    console.log('1. 发送验证码...');
    const smsResult = await sendSmsCode('CN', '13800138000');
    console.log('验证码发送结果:', smsResult);
    
    // 2. 用户注册（需要用户输入收到的验证码）
    console.log('2. 用户注册...');
    const registerResult = await registerUser('CN', '13800138000', '123456');
    console.log('注册结果:', registerResult);
    
    if (!registerResult.success) {
      throw new Error('注册失败');
    }
    
    const userId = registerResult.data;
    
    // 3. 获取 KYC 链接
    console.log('3. 获取 KYC 链接...');
    const kycResult = await getGuidanceLink(userId, 'KYC');
    console.log('KYC 链接:', kycResult);
    
    // 4. 查询用户资产
    console.log('4. 查询用户资产...');
    const balanceResult = await getUserBalance(userId);
    console.log('用户资产:', balanceResult);
    
    console.log('快速开始完成！');
    
  } catch (error) {
    console.error('快速开始失败:', error);
  }
}

// 运行快速开始
quickStart();
```

## 下一步

完成快速开始后，您可以：

1. [配置 WebSocket](../websocket/abount.md) 监听实时事件
2. [管理资产](../asset/abount.md) 处理充值和提现
4. [查看完整 API 文档](../../README.md) 了解所有功能

## 常见问题

### 签名验证失败
- 检查时间戳是否在 5 秒内
- 确认签名算法实现正确
- 验证 payload 构造是否正确

### 网络连接问题
- 确认域名和端口正确
- 检查 IP 是否在白名单中
- 验证 SSL 证书配置

### 业务逻辑错误
- 查看 [错误码文档](../../API%20Reference/error-code/readme.md)
- 检查请求参数格式
- 确认业务流程顺序
