# DCS Standard Authorization 开发指南

欢迎使用 DCS Standard Authorization 开发指南。本指南将帮助您快速集成 DCS 的 PayFi 功能。

## 快速开始

- [产品介绍](home/Introduction.md) - 了解 DCS Standard Authorization 平台和核心功能
- [环境配置](home/environments.md) - 配置开发和生产环境
- [快速开始](home/quickstart.md) - 5分钟快速集成指南
- [Postman 配置](home/postmansetup.md) - API 测试工具配置

## 功能模块

### 账户管理
- [账户功能概述](account/abount.md) - 用户注册、KYC、身份验证

### 卡片管理
- [卡片功能概述](card/abount.md) - 卡片功能概述
- [卡片创建](card/create.md) - 卡片创建
- [卡片冻结/解冻](card/block.md) - 卡片状态管理
- [卡片状态](card/status.md) - 卡片状态

### 资产管理
- [资产功能概述](asset/abount.md) - 数字资产管理和查询

### 钱包功能
- [钱包功能概述](wallet/abount.md) - 充值、提现、地址管理

### 实时通信
- [WebSocket 概述](websocket/abount.md) - 实时数据推送和监听

## 环境信息

| 环境  | API 域名                    | WebSocket 域名               |
|-----|---------------------------|----------------------------|
| 主网  | https://api.thedecard.com | wss://stream.thedecard.com |
| 测试网 | https://api.uatdcd.com    | wss://stream.uatdcd.com    |



[错误码说明](../API%20Reference/error-code/readme.md)
