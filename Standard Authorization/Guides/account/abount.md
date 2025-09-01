# 账户管理使用指南

本指南将帮助您快速了解和使用 DCS Standard Authorization 账户管理功能，包括用户注册、KYC认证和账户状态管理。

## 快速开始

### 用户注册流程

1. **发送短信验证码**
   - 使用 [验证码接口](../../API%20Reference/captcha/readme.md) 向用户手机发送验证码

2. **创建用户账户**
   - 调用 [用户注册接口](../../API%20Reference/account/readme.md#1-新建用户)，获得 DCS 用户ID
   - 支持手机号 AES 加密传输

3. **完成 KYC 认证**
   - 获取 [KYC 引导页链接](../../API%20Reference/account/readme.md#2-获取引导页链接)
   - 用户完成身份认证流程
   - [查询认证状态](../../API%20Reference/account/readme.md#4-查看用户kyc状态)

### 基本使用流程

1. **用户注册**：使用手机号和短信验证码创建账户，获得 DCS 用户ID
2. **KYC认证**：获取个性化的 KYC 认证链接，引导用户完成身份验证
3. **状态查询**：实时跟踪 KYC 认证进度和用户账户状态

详细的接口调用方法和参数说明，请参考 **[API 接口文档](../../API%20Reference/account/readme.md)**

## 主要功能

### 用户管理
- **用户注册**：通过手机号和短信验证码创建账户 → [详细说明](../../API%20Reference/account/readme.md#1-新建用户)
- **状态查询**：检查用户的各种状态标识 → [详细说明](../../API%20Reference/account/readme.md#3-查询用户状态)
  - `forbidWithdraw`：用户是否禁止提现
  - `forbidCardTransaction`：用户是否禁止卡交易

### KYC 认证
- **引导页生成**：为用户生成个性化的 KYC 认证链接 → [详细说明](../../API%20Reference/account/readme.md#2-获取引导页链接)
- **状态跟踪**：实时查询 KYC 认证进度和结果 → [详细说明](../../API%20Reference/account/readme.md#4-查看用户kyc状态)
- **KYC状态**：
  - `UNDO`：未做KYC
  - `INIT`：初始状态
  - `REVIEW`：提交申请
  - `PENDING`：上报 ANY Task，正在审核中
  - `PASS`：KYC通过
  - `REFUSE`：KYC拒绝

 KYC 状态发生变更时会收到 [KYC WebSocket](../../API%20Reference/websocket/readme.md#kyc状态-kyc_status) 通知

### Travel Rule 合规
- **信息查询**：获取用户的合规信息 → [详细说明](../../API%20Reference/account/readme.md#6-查询travel-rule)
- **信息更新**：维护大额转账所需的合规数据 → [详细说明](../../API%20Reference/account/readme.md#7-更新travel-rule)
- **必填信息**：
  - `name`：用户姓名
  - `dateOfBirth`：出生日期
  - `placeOfBirth`：出生国家（ISO 2位国家代码）
- **证件信息**：
  - `type`：证件类型（PASSPORT / ID_CARD）
  - `value`：证件号码
  - `countryOfIssue`：证件颁发国家（ISO 2位国家代码）
- **地址信息**：
  - `city`：城市
  - `postalCode`：邮编
  - `addressLines`：详细地址
  - `country`：国家（ISO 2位国家代码）
  - `region`：区域（可选）
  - `dependentLocality`：依赖地区（可选）
- **渠道信息**：
  - `channelName`：渠道名称
  - `institutionName`：机构名称（如：FOMO）


## 重要提醒

> [!WARNING]
> ### 注意事项
> - 手机号支持 AES 加密传输，建议在生产环境使用
> - 注册用户返回的用户id 需要妥善保存，这个id 用于后续 的操作。
> - KYC 链接有效期仅 5 分钟，需及时使用
> - 引导页会验证 referer 和 userAgent，确保参数正确

> [!CAUTION]
> ### 状态管理
> - KYC 状态会实时更新，建议 WebSocket 监听
> - 用户状态影响提现和卡交易功能，请关注 WebSocket 用户状态监听
> - Travel Rule 信息需要完整填写以满足合规要求