# ATH 协议知识库（内置精简版）

> 本版本为演示用内置知识库。可在「模型设置 → 知识库内容」中替换为完整版（20 章）。
> ATH = Agent Trust Handshake（智能体信任握手协议）：解决 Agent 之间"双向授权 + 细粒度信任"的协议。

## 1. 协议概述
ATH 是一个面向智能体（Agent）之间双向授权与信任握手的开放协议。传统 OAuth 2.0 只解决"用户同意授权"，不验证"对方 Agent 是否可信"。ATH 在 OAuth 基础上增加双向能力：客户端 Agent 与服务端 Agent 在握手阶段互相验证身份、能力与信任边界，通过细粒度能力授权（capability-scoped）替代全量授权。

## 2. 核心概念
- DID（Decentralized Identifier）：Agent 的分布式身份标识，形如 did:ath:<id>。
- 能力清单（Capability List）：Agent 对外声明可被授权的操作集合，如 handshake:write、token:verify。
- nonce：一次性随机数，用于防止重放攻击，握手请求必须携带。
- 信任握手（Trust Handshake）：客户端与服务端建立双向信任的完整流程。
- 细粒度授权：按"能力 + 资源 + 范围"授权，最小权限原则。

## 3. 握手流程（handshake 端点）
1. 客户端构造握手请求：DID、公钥、能力清单、nonce。
2. 服务端验证客户端 DID 与公钥（可选：通过信任锚或第三方验证）。
3. 服务端返回握手响应：服务端 DID、公钥、支持的能力子集、会话令牌。
4. 双方各自校验对方能力清单，握手完成。

## 4. 能力发现（discovery 端点）
客户端通过 GET /.well-known/ath-configuration 获取服务端支持的协议版本、能力清单、扩展。未声明的能力视为不支持。

## 5. 令牌（token 端点）
ATH 使用 JWT（RFC 7519）作为会话令牌。签发与校验必须包含三项：
- audience（受众）：必须匹配服务端配置的预期值（如 ath:app）。
- scope（范围）：令牌只能携带握手时协商的能力范围。
- expiry（过期）：令牌必须有过期时间，服务端必须校验。
若三项任一不匹配，服务端必须拒绝并返回 401（INVALID_AUDIENCE / INVALID_SCOPE / TOKEN_EXPIRED）。

## 6. 验证（verify 端点）
资源提供方通过 verify 端点校验令牌的有效性与权限，返回该令牌对应的能力、受众、过期时间。校验失败返回 401 错误码。

## 7. 错误码表（节选）
- 400 INVALID_PARAMS：请求参数缺失或格式错误。
- 401 INVALID_AUDIENCE：令牌 audience 与服务端期望不匹配。
- 401 INVALID_SCOPE：令牌 scope 超出握手协商的能力。
- 401 TOKEN_EXPIRED：令牌已过期。
- 401 UNSIGNED_TOKEN：服务端不接受未签名令牌。
- 404 DISCOVERY_FAILED：能力发现端点不可达。
- 409 HANDSHAKE_CONFLICT：nonce 重复或握手状态冲突。

## 8. 安全要求
- 客户端 MUST 在握手请求中携带 nonce 与公钥。
- 服务端 MUST NOT 接受未签名令牌，即使声明为内部调用。
- 服务端 SHOULD 校验令牌的 audience、scope、expiry 三项，缺一不可。
- 客户端 SHOULD 在握手前查询服务端能力清单。
- 令牌生命周期内，能力变更 SHOULD 触发重新握手。

## 9. 参考标准
OAuth 2.0（RFC 6749）/ PKCE（RFC 7636）/ RFC 8707（资源指示）/ RFC 7519（JWT）/ RFC 2119（关键词约定）/ MCP（Model Context Protocol）/ A2A（Agent-to-Agent）。

## 10. 术语表（中英文对照）
- 握手 Handshake / 能力清单 Capability List / 信任锚 Trust Anchor / 受众 Audience / 范围 Scope / 令牌 Token / 刷新 Refresh / 撤销 Revoke / 发现 Discovery / 验证 Verify / 随机数 Nonce / 公钥 Public Key / 私钥 Private Key / 签名 Sign / 重放攻击 Replay Attack / 中间人 MITM / 双向授权 Mutual Authorization / 细粒度 Fine-grained / 最小权限 Least Privilege / 会话 Session / 吊销 Revocation