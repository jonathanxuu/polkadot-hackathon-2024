# 为 Polkadot 生态系统带来 Mass Adoption 的基础设施 —— zkLogin

## 1. 现状
### 背景

账户抽象（Account Abstraction，简称 AA）是一种创新概念，旨在解决区块链上管理加密密钥所面临的挑战。区块链上的账户可以代表从个人身份到机构的各种实体，在 Web3 时代发挥着至关重要的作用，用户通过私钥进行数字签名交易，交易数据则记录在公共账本上。

然而，私钥的丢失会带来重大风险，导致数字资产无法访问和数字身份的碎片化。这迫使用户必须创建一个新账户并生成一组新的密钥，这在安全性和用户接受度上对用户和开发者都造成了不小的障碍。

账户抽象的概念最早在以太坊的 [EIP-4337](https://eips.ethereum.org/EIPS/eip-4337) 中提出，旨在为用户提供灵活性，使其能够在账户中编程以增强安全性和改善用户体验。这种将用户体验与私钥分离的方式，使得代码能够决定账户的行为，增加了账户的灵活性，减少了由于密钥管理不善造成的风险。



### 账户抽象现状
当前所有 Web3 生态系统都有一个共同目标：吸引大量 Web2 用户，实现 "Mass Adoption"。而账户抽象（Account Abstraction，简称 AA）是实现这一目标的关键。AA 旨在简化 Web3 应用中的用户体验，使用户能够在不必担心资产托管、钱包或区块链复杂性等问题的情况下与各种应用进行交互。用户在使用过程中，将面对一个无比熟悉且友好的界面。

像 `TON` 和 `STARKNET` 等热门区块链，已经在架构层面对 AA 提供了原生且无缝的支持。然而，`Polkadot` 生态系统中 AA 的发展略显滞后。相比之下，Ethereum 生态系统已经探索了多种新型账户管理的方法，包括"社交登录（Social Login）"、"Passkey" 和"电子邮件登录(Email Login)"。这些努力取得了显著成果，成功吸引了数千万用户，并推动着 AA 的发展。

![AA 相关数据](https://hackmd.io/_uploads/S1Uhpvm1Jg.png)
目前，账户抽象的使用正呈现出"指数级增长"，无论是交易量还是用户数量都在迅速增加，这证明了账户抽象在推动 Web3 下一阶段 —— 用户扩展中的潜力。
> 有关账户抽象（AA）更多统计数据，请查看[此链接](https://dune.com/sixdegree/account-abstraction-overview)。
> 
![用户量的增长](https://hackmd.io/_uploads/BkDX0w7yJe.png)

## 2. 什么是 zkLogin？ - 为 Polkadot 生态带来更简单、安全的账户抽象

`zkLogin` 解决方案基于 `WebAuthn` 技术。它以 Runtime Pallet 的形式存在，无需对任何 Polkadot 现有架构进行修改，任何项目都可通过添加 Runtime Pallet 的方式接入 zkLogin。
> `WebAuthn` 是一种现代身份验证标准，提供强大的身份验证功能。在 `Polkadot` 生态系统中集成这种身份验证特性将显著增强安全性，并为用户提供更多的认证选项。

具体而言，它允许用户跳过与外部拥有账户（EOAs）相关的传统繁琐但非常重要登录过程 —— 完全摆脱对助记词、私钥的依赖。即，`zkLogin` 使用 `Google` 账户作为用户身份验证的媒介，通过证明对 `Google` 账户的拥有权来验证相应链上地址的所有权。

![Google 和 zkLogin 地址](https://hackmd.io/_uploads/BkauL_Qkkl.png)

由于钱包复杂性导致的新用户入门障碍，是区块链中一个长期存在的问题，而 `zkLogin` 是其完美的解决方案。通过使用前沿的密码学和技术，`zkLogin` 既优雅又可靠。

在创建传统的区块链地址时，地址将关联一个永久的公钥-私钥对，并且地址标识符是从公钥派生的。而使用 zkLogin，zkLogin 地址由 `Google JWT` 和用户唯一的 `user_salt` 派生而来，用户在交易全程对私钥无感，用户的身份由 Google 提供 OAuth 验证服务。

简而言之，zkLogin 的流程可以大致分为以下几步：
![简单流程图](https://hackmd.io/_uploads/r1vIzTQJkx.png)

- 第一步：生成临时密钥
当用户打开应用时，它会为用户生成一个临时密钥对，这个密钥对有一个有效期限，决定了所需登录的频率。
- 第二步：向 Google 请求 JWT
应用提示用户通过 OAuth 提供商(如 `Google`)登录来进行身份验证，生成一个JSON Web Token（JWT），这是提供商的数字签名数据的有效载荷。在 zkLogin 中，这个 JWT 包括一个 nonce 字段，其中包含了公钥以及一个过期时间。JWT 中嵌入了密钥声明，用于后续生成 Polkadot 地址。
- 第三步：请求用户的唯一 salt
salt（用于随机数据的常见加密术语）是一个任意的数字字符串，与 JWT 和临时密钥对一起用来将 OAuth 凭据与 Polkadot 地址关联起来。
- 第四步：生成零知识证明
凭借JWT token、salt和公钥，流程继续请求从零知识（zk）证明服务中获得zk证明。证明其用户身份已由 google 验证通过，证明用户临时密钥的有效性。
- 第五步：识别用户的 Polkadot 地址并构建交易
然后，应用程序根据从 JWT 中提取的用户 salt 和密钥声明来识别用户的 Polkadot 地址。在提交过程中，提交与临时签名、zk证明和JWT的私钥捆绑的附加输入。
- 第六步：验证交易
一旦在区块链上，zkLogin Pallet 将仔细检查 zk 证明和临时签名以进行认证，无缝地完成整个流程。





**从隐私角度看**
- `Google` 账户与 `zkLogin 地址`之间没有公开链接。
- 除了用户本人，任何人都无法知道用户的 `zkLogin 地址`，包括 `Google`。（这是因为在从 Google 账户转换为 zkLogin 账户的过程中需要使用一个唯一的盐值。这个盐值是私密的，只有用户自己持有，因此即使是 Google 也无法确定用户的链上地址。）

**从安全角度看**
- 用户不再需要担心因私钥丢失而导致无法访问钱包的情况（传统外部账户（EOAs）中常见的安全性问题）
- 用户唯一可能失去资金访问权限的情况是：在过期时间内，他们同时丢失了"临时私钥"、JWT Token 和盐值。

集成 zkLogin 将成为 Polkadot 生态系统未来发展的一个很有前景的方向，它有助于提升用户数字身份的安全性和便捷性。通过消除对密码的依赖，`zkLogin` 减少了网络钓鱼、暴力破解和重放攻击的风险，为 Web2 和 Web3 用户提供了更简单、更快捷的登录体验。


### 与传统 EOA 的性能比较


| 对比方向 | zkLogin | 传统 EOA |
| -------- | -------- | -------- |
| 隐私保护     | 利用**零知识证明**进行用户身份验证，而**不暴露任何敏感信息**，如密码或私钥。这确保了用户的身份和个人数据在身份验证过程中保持私密。     | 依赖公私钥对进行身份验证。虽然公钥在区块链上是可见的，但私钥必须保密。如果用户的私钥被泄露，他们的账户和资产可能会受到威胁。     |
| 安全性 | 设计上能够抵御重放攻击和社交工程攻击。由于它**不需要共享敏感信息**，攻击者无法轻易利用用户凭证，即使他们拦截了身份验证数据。| 容易受到各种攻击，包括钓鱼和密钥泄露。如果攻击者获得了用户的私钥，他们可以完全控制相关账户和资产。用户必须谨慎管理他们的私钥以降低风险。 |
| 何种情况下账户将被盗用 | 在设定的“**过期时间**”内，用户**同时丢失**"**临时私钥**"、**JWT Token** 和**盐值** | 私钥或助记词丢失 |
| 用户体验 | 提供**无私钥登录**体验，简化了身份验证过程。用户无需记住复杂的密码，一切都如同 Web2 世界，从而提高了便利性，并降低了与密码相关的安全漏洞风险。| 用户负责管理他们的私钥和恢复短语，这可能比较繁琐。对这些密钥的错误管理或丢失可能导致无法恢复的资金访问丧失。 |
| 可拓展性 | 支持跨多个应用程序的共享身份验证。这意味着用户可以在不同服务之间进行身份验证，而无需反复输入敏感信息，从而简化了整体用户体验。 | 通常需要为每个服务进行单独身份验证，增加了管理复杂性。用户必须处理多个密钥，这可能在安全维护上带来挑战。 |



## 3. 对 Polkadot 生态的主要价值

### 1. 无缝集成 Relaychains & Parachains

`zkLogin` 作为一个 Runtime Pallet，使得任何 relaychain 或 parachain 都可以通过简单地添加此 runtime pallet 来直接与 zkLogin 进行交互。Polkadot 生态共享一个统一的代码库，为引入 zkLogin 解决方案提供了独特的优势。

### 2. 完全兼容 polkadot{.js}

`zkLogin` 交易体的构建与原生 Polkadot 交易体相同。只需添加 zkLogin pallet，用户即可在不干扰 Polkadot 链上交易架构的情况下使用它。这对于现有工具非常友好，任何用于构建或发起 Polkadot 交易的当前工具（如`polkadot{.js}`）都无需进行任何修改，极大地提升了开发者的友好性。

### 3. 增强 Polkadot 生态系统的隐私和安全性

基于零知识证明的 `zkLogin` 解决方案，为用户提供了更高水平的隐私和安全性。用户的认证信息无需以明文存储在区块链上，从而有效保护用户隐私。

### 4. 促进 Polkadot 生态系统的 Mass Adoption

`zkLogin` 的引入显著提升了用户与 Polkadot 及其 parachain 互动的可及性，降低了用户的进入门槛。用户可以轻松、安全地管理他们的认证信息，提升整体用户体验。
这也减少了 DApp 开发者的开发复杂性，使任何项目都能快速集成 zkLogin，并吸引大量 Web2 用户。

### 5. 弥补 Polkadot 生态系统"零知识技术"和"账户抽象"应用的缺乏

目前，Polkadot 生态系统中缺少利用"零知识技术"和"账户抽象（AA）技术"的具体且引人注目的应用场景，使得 Polkadot 在创新和用户体验提升方面处于相对滞后的状态。零知识技术作为保护用户隐私和提高交易效率的重要工具，在其他区块链生态中已被证明有巨大潜力，而 Polkadot 目前却没有充分挖掘这一技术的实际应用。
此外，账户抽象技术可以显著简化用户与区块链的交互流程，提升用户体验，特别是对于新用户和 Web2 用户的引入具有重要作用，这也是当前 Polkadot 生态在用户体验和用户吸引方面欠缺的。


## 4. zkLogin 技术方案细节

`zkLogin` 为用户提供了一种通过 `OAuth 凭证` 从 Polkadot `zkLogin 地址`发送交易的能力，而无需对外暴露两者之间的任何联系。

这是将用户引入区块链最简单的方式之一。`zkLogin` 允许用户使用现有的 Web2 认证提供商（如 `Google`）登录 Web3 应用程序，省去了用户记住或记录私钥的麻烦。

`zkLogin` 在不牺牲安全性的前提下为终端用户带来了极大的便利。它通过临时密钥对和零知识加密技术，将 Web2 认证提供商的 OAuth 凭证与特定的 Polkadot 账户连接起来。使用 `zkLogin` 时，提交到区块链的唯一数据是零知识证明、临时签名和一些辅助数据，完全不需要向区块链提交任何用户信息。此外，Web2 认证提供商也不会察觉到用户正在使用区块链，从而确保了用户的隐私安全。



### 为什么我们使用零知识？
使用零知识证明（ZKPs）旨在确保**不暴露任何用户的隐私数据**。在零知识证明框架内完成的任务包括：
1. 验证提供给 `Google` 进行身份验证的 `nonce` 是否符合 `zkLogin` 规则，具体为：
```js=
jwt_token.payload.nonce = H(eph_pk, max_epoch, jwt_randomness).
```
2. 验证用户的 JWT 令牌：证明该 JWT 令牌确实是由 `Google` 签署的。
3. 确认用户的 `zkLogin` 地址是否符合生成规则，计算方式为：
```js=
address = H(jwt_token.payload.iss, H(user_salt, jwt_token.payload.aud, jwt_token.payload.sub)).
```
这些计算涉及用户的私密数据，均**在用户设备上本地执行**。区块链上仅提交零知识证明用于验证，从而有效防止恶意的身份冒充攻击。
![上链需提交的内容](https://hackmd.io/_uploads/rkRGwt7kyx.png)

零知识证明规定，只有**特定的临时密钥**在**指定的时间范围**内（`expirationTime` 之前）有权操作给定的 `zkLogin 地址`。随后，当用户执行交易时，他们需要提供“由临时密钥签署的签名”以及“零知识证明”。


### Detailed WorkFlow

![Detailed zkLogin WorkFlow](./detailed_workflow.png)
- (步骤 0) 我们在协议的 zkSNARK 实例化中使用 Groth16，需要生成与电路相关联的结构化公共参考字符串 (CRS)。进行仪式生成 CRS，用于在 ZK 证明服务中生成证明密钥并生成验证密钥。
- (步骤 1-3) 用户首先通过登录到 OpenID 提供程序 (OP) 来获取包含定义的 `nonce` 的 `JWT 令牌`。在此过程中，用户会生成一个临时密钥对 (`eph_sk`, `eph_pk`)，同时生成到期时间 (`max_epoch`) 和随机数 (`jwt_randomness`)。通过这些参数，钱包扩展可以计算 `nonce`。当用户完成 OAuth 登录流程后，将检索到 JWT 令牌。
- (步骤 4-5) 然后，应用程序前端将 JWT 令牌发送到 `salt 服务`。`salt 服务`在验证 JWT 令牌后，基于 `iss`, `aud`, `sub` 返回唯一的 `user_salt`。
- (步骤 6-7) 钱包将 JWT 令牌、`user_salt`、`eph_pk`、`jwt_randomness` 及其他一些输入数据传递至 ZK 生成服务。ZK 证明服务根据这些私密输入生成一个零知识证明，并执行以下任务：
    - a) 检查 `nonce` 是否根据规则正确派生；
    - b) 检查密钥声明值是否与 JWT 中的对应字段匹配；
    - c) 验证 JWT 的 RSA 签名是否由指定 OP 签署；
    - d) 验证生成的地址与是否一致。
- (步骤 8): 钱包基于 `iss`、`aud`、`sub` 及 `aud` 计算用户地址。只要钱包扩展持有有效的 JWT 令牌，此步骤可以独立完成。
- (步骤 9-10) 使用临时私钥签署交易以生成临时签名。最后，用户将交易与临时签名、ZK 证明和其他输入一起提交至区块链。
- (步骤 10 之后) 在链上提交后，我们基于 Substrate 框架验证该零知识证明，确保其与存储中提供者的 JWK（通过共识达成）相符，同时验证临时签名。

通过该流程，用户可以安全且隐私地通过 OAuth 提供商进行登录并与区块链交互，无需公开关联其链上地址和 Web2 账户。



### 前端实现

在创建用户友好且无缝的应用界面的过程中，前端实现扮演着至关重要的角色。zkLogin 的前端设计特别强调打造一个"让用户无感知交互"的环境。

用户将直接体验到流畅而直观的界面 —— 无需主动管理加密密钥，无需进行复杂的身份验证流程，一切正如 web2 世界里常见的那样，一键 Google 登陆，完成。这样的设计旨在使用户能够轻松自如地使用应用程序，从而降低技术门槛，促进更广泛的用户参与。



#### 1. 安全的 Polkadot zkLogin 账户生成

为了增强 Polkadot 账户的安全性，前端将结合加密算法，确保密钥生成的强健性。该过程涉及使用复杂的加密技术，以在 Polkadot 生态系统内创建和管理安全的用户账户。

**解决方案:**

在账户生成生成阶段，钱包应能够在 SDK 内完成以下任务：

1. 生成并存储一个临时密钥对（`eph_sk`，`eph_pk`）。遵循传统钱包生成密钥对的相同过程。
2. 设置临时密钥对的过期时间。钱包决定最大时间是否为当前时间或更晚，同时钱包也可以决定用户是否可以调整此时间。
3. 为当前会话生成 JWT 随机数（`jwt_randomness`）。

```ts
// Example Code for demonstration
onst maxBlocknumber = Number(now) + 2000; // this means the ephemeral key will be active for 2000 blocknumbers from now.
const ephemeralKeyPair = new Ed25519Keypair();
const jwt_randomness = generateRandomness();
const nonce = generateNonce(ephemeralKeyPair.getPublicKey(), maxBlocknumber, jwt_randomness);
```


#### 2. 集成身份验证平台

前端将具备与身份验证平台无缝集成的能力，从而实现流畅的用户登录体验。通过与这些平台（如 `Google`）的接口，应用程序确保用户可以轻松进行身份验证，利用行业标准的 OAuth 协议。

**解决方案**: 

1. 登录、获取和解码 JWT 令牌：
当用户使用 zkLogin 通过身份验证提供商（如 `Google`）登录时，身份验证提供商将以 URL 参数的形式返回 JWT 令牌，其构造方式及具体字段如下：
```ts
const decodedJwt = jwt_decode(encodedJWT) as JwtPayload;

export interface JwtPayload {
   iss?: string;
   sub?: string;  //Subject ID
   aud?: string[] | string;
   exp?: number;
   nbf?: number;
   iat?: number;
   jti?: string;
}
```
2. 生成或恢复 `user_salt`：
在获取 JWT 令牌 后，钱包会在本地生成一个称为 user_salt 的随机数（或由用户确定，若用户非首次创建将从"用户 Salt 备份服务"中取回用户 user_salt）。

3. 构造用户账户地址：
```ts
// jwtToAddress logic: address = H(iss, H(user_salt, aud, sub))
const zkLoginUserAddress = jwtToAddress(jwt, userSalt);
```
对于用户而言，(`iss`, `aud`, `sub`) 是固定的，因此地址基本上由 `user_salt` 决定。不同的 `user_salt` 值会导致不同的地址。
在此过程中，存储 `user_salt` 至关重要。我们可以通过我们的 Salt Backup Service 以以下可选方法进行备份（待确定）：
- 客户端：
    - 选项 1：在访问钱包时请求用户输入盐值，将责任转移给用户，用户必须记住它。
    - 选项 2：浏览器或移动设备存储：确保合适的工作流程，以防止用户在设备或浏览器更改时丢失钱包访问权限。一种方法是在新钱包设置期间通过电子邮件发送盐值。
- 后端服务：
    - 选项 3：在传统数据库（例如用户或密码表）中存储用户标识符（例如 `sub`）到用户盐的映射。每个用户的盐值是唯一的。
    - 选项 4：实现一个服务，保持一个主种子值，并通过验证和解析 JWT 令牌来推导用户盐。例如，使用 HKDF(ikm = seed, salt = iss || aud, info = sub) 进行定义。请注意，此选项不允许主种子或客户端 ID（即 aud）更新，否则将推导出不同的用户地址，并导致资金丢失。

#### 3. 零知识证明生成
该服务负责处理 Google 提供的 JWT令牌，并能对这些信息及用户的秘密信息执行零知识证明。这种先进的加密功能确保了用户数据在身份验证过程中的隐私和完整性，为用户账户和交互提供了安全层。

**解决方案**

钱包扩展应嵌入一个 ZK 证明生成服务，以便利用 `groth16` 证明系统在本地生成 zk 证明。这是对临时密钥对的证明（证明），用于验证该临时密钥对的有效性。之后用户使用该临时密钥对即可代表某一 Polkadot zkLogin 账户发起交易。


#### 4. zkLogin 交易集成

前端将具备通过 `zkLogin` **发起交易**的能力。利用零知识证明，应用程序可以在不暴露敏感信息的情况下对用户**进行身份验证并授权交易**。此隐私保护特性增强了用户交互的整体安全性和可信度。

**解决方案**

1. 通过**临时密钥**签署交易：
首先，使用之前生成的密钥对中的**临时私钥对**交易字节进行**签名**。这与传统的密钥对签名方式相同。

```ts
const ephemeralKeyPair = new Ed25519Keypair();

// ....
const unsigned = methods.abcd({...}, {...}, {...});
const signingPayload = createSigningPayload(unsigned, { registry });

const userSignature = await signWithEphemeralKey(signingPayload, ephemeralKeyPair);
```

2. 将 ZK 证明与之前的签名结合：
在此过程中，我们将之前获得的"**零知识证明**"与用户使用临时密钥对进行的"**交易签名**"结合起来，从而生成交易的最终签名（`zkLoginSignature`）。
```ts
const addressSeed : string = genAddressSeed(BigInt(userSalt!), "sub", decodedJwt.sub, decodedJwt.aud).toString();

const zkLoginSignature : SerializedSignature = getZkAuthignature({
   inputs: {
      ...ZKProofSig,
      addressSeed
   },
   maxEpoch,
   userSignature,
});
```

3. 提交交易:
```ts
client.executeTransactionBlock({
    tx: unsigned,
    signature: zkLoginSignature,
});
```

### 后端实现

#### 1. 基于 Substrate 框架的集成
- Substrate 框架
后端需要基于 Substrate 框架进行开发，以确保与 zkLogin 的兼容性。这涉及对现有组件和功能进行调整，最终实现 zkLogin 以 runtime Pallet 的形式适配于各类 parachain 及 relaychain，而无需对链原有结构进行任何修改。
- 高可移植性代码
该项目旨在在整个 Polkadot 生态系统中实现通用使用。我们优先考虑后端的稳健性和可移植性，这一生态友好的解决方案将极易于扩展到各种 relaychain 和parachain。

#### 2. 零知识证明验证
基于前端 zkproof 生成阶段所使用的零知识算法和数字电路，需要在后端实现高效的验证模式，优先考虑效率和准确性。后端确保一个稳健且简化的验证过程，有助于提高 zkLogin 系统的整体效率和可靠性。