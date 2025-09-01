# 交易

## 概述
DCS交易系统基于预授权和实际交易分离的设计理念，通过Authorisation进行资金预冻结，通过Transaction记录实际资金变动，最终通过Outstanding生成统一账单。

---

## 核心交易概念

### 🔒 Authorisation（预授权）

预授权是用户消费时的授权请求，对应资金的冻结和解冻操作，但不产生实际资金扣除。

#### Direction（交易方向）
| 值 | 含义 | 资金流向 |
|---|---|---|
| `OUTGOING` | 用户付款，冻结余额账户（合作商账户） | 无真实资金流动 |
| `INCOMING` | 用户收款，解冻余额账户（合作商账户） | 无真实资金流动 |

#### 主要字段说明
- **auth_id**: 授权唯一流水号
- **direction**: 交易方向（OUTGOING/INCOMING）
- **amount**: 授权金额
- **original_auth_id**: 原始授权交易唯一流水号（用于关联原交易）
- **approve_status**: 授权状态（A=已批准）
- **outs_id**: 关联的Outstanding ID
- **type**: 授权类型（pre-auth预授权、pre-auth-completion预授权完成等）

#### 业务流程
1. **用户付款场景**：
   - 收到OUTGOING的Authorisation → 冻结用户对应资产
   
2. **用户收款场景**：
   - 收到INCOMING的Authorisation → 解冻用户对应资产
   - 如果用户冻结资产小于要解冻的资产，可拒绝此次Authorisation

---

### 💰 Transaction（交易流水）

Transaction记录实际的资金扣除和增加，不记录冻结和解冻操作。

#### Direction（交易方向）
| 值 | 含义 | 资金流向 |
|---|---|---|
| `OUTGOING` | 用户付款 | 从余额账户（合作商账户）出金 |
| `INCOMING` | 用户收款 | 向余额账户（合作商账户）入金 |

#### 主要字段说明
- **transaction_id**: 交易唯一标识
- **direction**: 交易方向（OUTGOING/INCOMING）
- **amount**: 实际交易金额
- **auth_id**: 关联的授权ID
- **outs_id**: 关联的Outstanding ID

#### 业务场景
1. **出金**：清分时，从余额账户（合作商账户）出金
2. **入金**：清分时，向余额账户（合作商账户）入金

---

### 📊 Outstanding（账单）

Outstanding统计Authorisation和Transaction的综合账单，多个预授权和多个交易可以组合成一个Outstanding。

#### 主要字段说明
- **outs_id**: 账单唯一标识
- **auth_amount**: 当前预授权冻结金额
- **amount**: 账单总金额


## 重要业务规则
### 资金处理原则
1. **预授权阶段**：只进行资金冻结/解冻，不产生实际扣款
2. **清分阶段**：通过Transaction记录实际资金变动
3. **账单生成**：通过Outstanding统一管理多笔关联交易

---

## 数据流转

```
预授权请求 → 资金冻结 → 预授权批准 → 清分处理 → Transaction生成 → 账单更新 → 对账文件推送
```

该交易体系确保了资金安全性和数据一致性，支持各种复杂的支付场景和异常处理。