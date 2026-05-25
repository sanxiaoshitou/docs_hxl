# Four.Meme 平台 API 说明文档

## 概述

本文档详细说明了 Four.Meme 平台的 API 接口和智能合约协议，包括代币创建、交易、查询等功能。

---

## 一、REST API 接口

### 1. 用户认证相关

#### 1.1 生成 Nonce
**接口**: `POST https://four.meme/meme-api/v1/private/user/nonce/generate`

**请求参数**:
```json
{
  "accountAddress": "用户钱包地址",
  "verifyType": "LOGIN",
  "networkCode": "BSC"
}
```

**响应**:
```json
{
  "code": "0",
  "data": "生成的nonce值"
}
```

#### 1.2 用户登录
**接口**: `POST https://four.meme/meme-api/v1/private/user/login/dex`

**请求参数**:
```json
{
  "region": "WEB",
  "langType": "EN",
  "loginIp": "",
  "inviteCode": "",
  "verifyInfo": {
    "address": "用户钱包地址",
    "networkCode": "BSC",
    "signature": "使用私钥签名的 'You are sign in Meme {nonce}'",
    "verifyType": "LOGIN"
  },
  "walletName": "MetaMask"
}
```

**响应**:
```json
{
  "code": "0",
  "data": "access_token"
}
```

---

### 2. 代币管理相关

#### 2.1 上传代币图片
**接口**: `POST https://four.meme/meme-api/v1/private/token/upload`

**Headers**:
```
Content-Type: multipart/form-data
meme-web-access: {access_token}
```

**参数**:
- `file`: 图片文件（支持 jpeg, png, gif, bmp, webp 格式）

**响应**:
```json
{
  "code": "0",
  "data": "上传后的图片URL"
}
```

#### 2.2 创建代币
**接口**: `POST https://four.meme/meme-api/v1/private/token/create`

**Headers**:
```
meme-web-access: {access_token}
Content-Type: application/json
```

**请求体示例**:
```json
{
  "name": "RELEASE",
  "shortName": "RELS",
  "desc": "RELEASE DESC",
  "imgUrl": "https://static.four.meme/market/...",
  "launchTime": 1740708849097,
  "label": "AI",
  "lpTradingFee": 0.0025,
  "webUrl": "https://example.com",
  "twitterUrl": "https://x.com/example",
  "telegramUrl": "https://telegram.com/example",
  "preSale": "0.1",
  "onlyMPC": false,
  "feePlan": false,
  "tokenTaxInfo": {
    "burnRate": 20,
    "divideRate": 30,
    "feeRate": 5,
    "liquidityRate": 40,
    "minSharing": 100000,
    "recipientAddress": "0x1234567890123456789012345678901234567890",
    "recipientRate": 10
  },
  "raisedToken": {
    "symbol": "BNB",
    "nativeSymbol": "BNB",
    "symbolAddress": "0xbb4cdb9cbd36b01bd1cbaebf2de08d9173bc095c",
    "deployCost": "0",
    "buyFee": "0.01",
    "sellFee": "0.01",
    "minTradeFee": "0",
    "b0Amount": "8",
    "totalBAmount": "24",
    "totalAmount": "1000000000",
    "logoUrl": "https://static.four.meme/market/...",
    "tradeLevel": ["0.1", "0.5", "1"],
    "status": "PUBLISH",
    "buyTokenLink": "https://pancakeswap.finance/swap",
    "reservedNumber": 10,
    "saleRate": "0.8",
    "networkCode": "BSC",
    "platform": "MEME"
  }
}
```

**响应**:
```json
{
  "code": "0",
  "data": {
    "createArg": "用于区块链的编码参数",
    "signature": "区块链交易的签名"
  }
}
```

#### 2.3 获取代币信息
**接口**: `GET https://four.meme/meme-api/v1/private/token/get?address={token_address}`

**或**: `GET https://four.meme/meme-api/v1/private/token/getById?id={requestId}`

**响应示例**:
```json
{
  "code": "0",
  "data": {
    "version": "V8",
    "aiCreator": true,
    "feePlan": true,
    "taxInfo": {
      "feeRate": 10,
      "recipientRate": 0,
      "burnRate": 0,
      "divideRate": 0,
      "liquidityRate": 100,
      "recipientAddress": "",
      "minSharing": 0,
      "m": 0,
      "e": 0
    }
  }
}
```

---

## 二、智能合约接口

### 1. TokenManager2 (V2) - 代币管理器

**合约地址 (BSC)**: `0x5c952063c7fc8610FFDB798152D69F0B9550762b`

**ABI 文件**: `TokenManager2.lite.abi`

#### 主要方法

##### 购买代币
```solidity
// 按金额购买
function buyTokenAMAP(address token, uint256 funds, uint256 minAmount) payable

// 按数量购买
function buyToken(address token, address to, uint256 amount, uint256 maxFunds) payable

// X Mode 专用购买方法
function buyToken(bytes args, uint256 time, bytes signature) payable
```

##### 出售代币
```solidity
// 基本出售
function sellToken(address token, uint256 amount)

// 带手续费接收者的出售
function sellToken(uint256 origin, address token, uint256 amount, 
                   uint256 minFunds, uint256 feeRate, address feeRecipient)
```

##### 创建代币
```solidity
function createToken(bytes createArg, bytes signature)
```

#### 主要事件

```solidity
// 代币创建事件
event TokenCreate(
    address creator,
    address token,
    uint256 requestId,
    string name,
    string symbol,
    uint256 totalSupply,
    uint256 launchTime,
    uint256 launchFee
)

// 代币购买事件
event TokenPurchase(
    address token,
    address account,
    uint256 price,
    uint256 amount,
    uint256 cost,
    uint256 fee,
    uint256 offers,
    uint256 funds
)

// 代币出售事件
event TokenSale(
    address token,
    address account,
    uint256 price,
    uint256 amount,
    uint256 cost,
    uint256 fee,
    uint256 offers,
    uint256 funds
)

// 流动性添加事件
event LiquidityAdded(
    address base,
    uint256 offers,
    address quote,
    uint256 funds
)
```

---

### 2. TokenManagerHelper3 (V3) - 代币助手

**合约地址**:
- BSC: `0xF251F83e40a78868FcfA3FA4599Dad6494E46034`
- Arbitrum One: `0x02287dc3CcA964a025DAaB1111135A46C10D3A57`
- Base: `0x1172FABbAc4Fe05f5a5Cebd8EBBC593A76c42399`

**ABI 文件**: `TokenManagerHelper3.abi`

#### 主要方法

##### 获取代币信息
```solidity
function getTokenInfo(address token) view returns (
    uint256 version,
    address tokenManager,
    address quote,
    uint256 lastPrice,
    uint256 tradingFeeRate,
    uint256 minTradingFee,
    uint256 launchTime,
    uint256 offers,
    uint256 maxOffers,
    uint256 funds,
    uint256 maxFunds,
    bool liquidityAdded
)
```

##### 预计算购买
```solidity
function tryBuy(address token, uint256 amount, uint256 funds) view returns (
    address tokenManager,
    address quote,
    uint256 estimatedAmount,
    uint256 estimatedCost,
    uint256 estimatedFee,
    uint256 amountMsgValue,
    uint256 amountApproval,
    uint256 amountFunds
)
```

##### 预计算出售
```solidity
function trySell(address token, uint256 amount) view returns (
    address tokenManager,
    address quote,
    uint256 funds,
    uint256 fee
)
```

---

### 3. TaxToken - 税费代币合约

**ABI 文件**: `TaxToken.abi`

#### 主要功能
- 交易费用机制：买卖交易收取一定比例的费用
- 奖励分配机制：代币持有者根据持有量获得奖励分配
- 多种分配模式：费用可分配给创始人、持有者、销毁和流动性

#### 关键状态变量
```solidity
uint256 public feeRate;          // 交易费率（基点，10000 = 100%）
uint256 public rateFounder;      // 创始人分配率
uint256 public rateHolder;       // 持有者分配率
uint256 public rateBurn;         // 销毁分配率
uint256 public rateLiquidity;    // 流动性分配率
uint256 public minShare;         // 最小持有量
```

#### 主要方法
```solidity
// 查询可领取奖励
function claimableFee(address account) view returns (uint256)

// 查询已领取奖励
function claimedFee(address account) view returns (uint256)

// 领取奖励
function claimFee() external

// 获取用户信息
function userInfo(address) view returns (
    uint256 share,
    uint256 rewardDebt,
    uint256 claimable,
    uint256 claimed
)
```

#### 主要事件
```solidity
// 费用分配事件
event FeeDispatched(
    uint256 amountFounder,
    uint256 amountHolder,
    uint256 amountBurn,
    uint256 amountLiquidity,
    uint256 quoteFounder,
    uint256 quoteHolder
)

// 奖励领取事件
event FeeClaimed(
    address account,
    uint256 amount
)
```

---

### 4. AgentIdentifier - AI代理识别器

**合约地址 (BSC)**: `0x09B44A633de9F9EBF6FB9Bdd5b5629d3DD2cef13`

**ABI 文件**: `AgentIdentifier.abi`

#### 主要方法
```solidity
// 检查是否为代理钱包
function isAgent(address wallet) external view returns (bool)

// 获取代理NFT数量
function nftCount() external view returns (uint256)

// 获取指定索引的代理NFT地址
function nftAt(uint256 index) external view returns (address)
```

---

## 三、代币类型识别

### 1. 识别 TaxToken（税费代币）

**链上方法**:
```javascript
const template = await tokenManager.methods._tokenInfos(tokenAddress).call();
const creatorType = (BigInt(template.template) >> BigInt(10)) & BigInt(0x3F);
const isTaxToken = creatorType === BigInt(5);
```

**链下方法**:
通过 API 查询，如果返回数据包含 `taxInfo` 对象，则为 TaxToken。

### 2. 识别 AntiSniperFeeMode（防狙击费用模式）

**链上方法**:
```javascript
const infoEx1 = await tokenManager.methods._tokenInfoEx1s(tokenAddress).call();
const isAntiSniper = BigInt(infoEx1.feeSetting) > BigInt(0);
```

**链下方法**:
通过 API 查询，检查 `feePlan` 字段，`true` 表示启用防狙击模式。

### 3. 识别 X Mode 独占代币

**链上方法**:
```javascript
const template = await tokenManager.methods._tokenInfos(tokenAddress).call();
const isExclusive = (BigInt(template.template) & BigInt(0x10000)) > BigInt(0);
```

**链下方法**:
通过 API 查询，如果 `version = V8`，则为独占代币。

### 4. 识别 AI 代理创建的代币

**链上方法**:
```javascript
const template = await tokenManager.methods._tokenInfos(tokenAddress).call();
const isCreatedByAgent = (BigInt(template.template) & (BigInt(1) << BigInt(85))) !== BigInt(0);
```

**链下方法**:
通过 API 查询，检查 `aiCreator` 字段，`true` 表示由 AI 代理创建。

---

## 四、固定参数说明

### 代币创建固定参数
- **totalSupply**: 1,000,000,000（10亿）
- **raisedAmount**: 24 BNB
- **saleRate**: 0.8（80%）
- **reserveRate**: 0
- **lpTradingFee**: 0.0025
- **symbol**: BNB（基础货币）

### tokenTaxInfo 参数约束
- **feeRate**: 只能是 1, 3, 5, 或 10（代表 1%, 3%, 5%, 10%）
- **burnRate + divideRate + liquidityRate + recipientRate = 100**
- **minSharing**: 必须满足 d × 10ⁿ（n ≥ 5, 1 ≤ d ≤ 9）

---

## 五、错误码

### 购买代币错误
- **GW**: 金额精度未对齐到 GWEI
- **ZA**: 'to' 地址不能为 address(0)
- **TO**: 'to' 地址不能设置为 PancakePair 地址
- **Slippage**: 购买花费超过 maxFunds
- **More BNB**: msg.value 中发送的 BNB 不足
- **A**: 使用了错误的方法购买 X Mode 独占代币

### 出售代币错误
- **GW**: 金额精度未对齐到 GWEI
- **FR**: 费率超过 5%
- **SO**: 订单金额太小
- **Slippage**: 出售收到的代币少于 minAmount

---

## 六、使用流程

### 完整代币创建流程
1. 调用 `/nonce/generate` 获取 nonce
2. 使用私钥签名消息并调用 `/login/dex` 获取 access_token
3. 调用 `/token/upload` 上传代币图片
4. 调用 `/token/create` 创建代币，获取 createArg 和 signature
5. 调用 TokenManager2 合约的 `createToken(createArg, signature)` 完成链上创建

### 代币交易流程
1. 调用 TokenManagerHelper3 的 `getTokenInfo(token)` 获取代币信息
2. 根据需要调用 `tryBuy` 或 `trySell` 预计算交易结果
3. 调用 TokenManager2 的相应方法进行实际交易

---

## 七、注意事项

1. 创建代币需要满足最低 BNB 余额要求（当前为 0.01 BNB）
2. 图片必须上传到 four.meme 平台
3. 大多数技术参数是固定的，不可调整
4. 代币标签必须是平台支持的类别之一
5. 使用正确的合约版本：V1 用于 2024年9月5日之前创建的代币，V2 用于之后创建的代币
6. 推荐使用 TokenManagerHelper3 (V3) 进行信息查询和预计算

---

**文档版本**: 1.0  
**更新日期**: 2026-05-19  
**参考文档**: API-Documents.03-03-2026.md, API-CreateToken.02-02-2026.md, API-Contract-TaxToken.md
