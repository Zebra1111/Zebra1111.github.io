## 前言

在开发博客系统时，我需要一个密钥认证机制来保护文章的发布和删除操作。虽然纯前端方案无法达到服务端认证的安全级别，但通过合理的设计，仍然可以实现一套实用的安全体系。

## 核心设计思路

### 为什么不能明文存储密码？

如果将密码以明文形式写在 JavaScript 代码中，任何人打开浏览器开发者工具就能直接看到密码。因此我们需要使用**单向哈希函数**——只存储密码的哈希值，验证时将用户输入进行哈希后与存储的哈希值比对。

> 即使攻击者获取了哈希值，也无法反推出原始密码

## 技术实现

### 1. SHA-256 哈希 (Web Crypto API)

我们使用浏览器原生的 Web Crypto API 来计算 SHA-256 哈希，无需引入任何第三方库：

```javascript
async function sha256(message) {
    const encoder = new TextEncoder();
    const data = encoder.encode(message);
    const hashBuffer = await crypto.subtle.digest('SHA-256', data);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
}
```

**关键点：**
- `TextEncoder` 将字符串转为 UTF-8 字节序列
- `crypto.subtle.digest` 是浏览器原生 API，性能优秀
- 最终输出 64 位十六进制字符串

### 2. 密码验证流程

验证过程是异步的，流程如下：

1. 用户输入密码
2. 调用 `sha256()` 计算输入密码的哈希
3. 将计算结果与预存的 `KEY_HASH` 常量比对
4. 匹配则认证成功，否则记录失败次数

```javascript
const KEY_HASH = '8d969e...'; // 预计算的哈希值

async function verifyPassword(password) {
    const hash = await sha256(password);
    return hash === KEY_HASH;
}
```

### 3. 防暴力破解机制

为防止攻击者通过自动化脚本穷举密码，系统实现了**失败次数限制 + 时间锁定**：

```javascript
const MAX_ATTEMPTS = 5;       // 最大尝试次数
const LOCKOUT_DURATION = 60000; // 锁定60秒
let failedAttempts = 0;
let lockoutUntil = 0;
```

**策略：**
- 连续失败 5 次后，锁定账户 60 秒
- 锁定期间任何验证请求直接拒绝
- 成功认证后重置失败计数器
- 显示剩余尝试次数，给用户明确反馈

### 4. 会话管理

认证成功后，系统创建一个**临时会话**，避免用户每次操作都需要输入密码：

```javascript
const SESSION_DURATION = 30 * 60 * 1000; // 30分钟

function startSessionTimer() {
    sessionTimer = setTimeout(() => {
        isAuthenticated = false;
        authSessionToken = null;
        updateAuthUI();
    }, SESSION_DURATION);
}
```

**特性：**
- 会话有效期 30 分钟，到期自动登出
- 使用 `crypto.randomUUID()` 生成随机会话令牌
- 提供手动登出功能
- 实时 UI 状态指示器（绿点 = 已认证）

### 5. 用户体验设计

认证弹窗采用了多种 UX 优化：

- **焦点管理**：弹窗打开自动聚焦密码输入框
- **键盘支持**：Enter 确认、Escape 取消
- **抖动动画**：密码错误时输入框抖动（CSS `shake` animation）
- **错误反馈**：实时显示错误信息和剩余尝试次数
- **状态指示器**：导航栏显示当前认证状态

```css
@keyframes shake {
    0%, 100% { transform: translateX(0); }
    20% { transform: translateX(-10px); }
    40% { transform: translateX(10px); }
    60% { transform: translateX(-6px); }
    80% { transform: translateX(6px); }
}
```

## 安全分析

### 优势
- 密码不以明文存储在代码中
- SHA-256 抗碰撞、抗预像攻击
- 防暴力破解的频率限制
- 会话自动过期机制

### 局限性
- 前端代码可被审查，哈希值可见（可针对性彩虹表攻击）
- 客户端逻辑可被绕过（通过修改 JS）
- 无服务端验证，安全性依赖客户端

### 可能的改进方向
- 引入 **PBKDF2/Argon2** 增加哈希计算成本
- 添加**盐值 (Salt)** 防止彩虹表攻击
- 使用后端 API 进行真正的服务端认证
- 结合 **JWT Token** 实现跨页面会话

## 总结

这套前端密钥系统在保证易用性的同时，尽可能地提升了安全性。虽然它不适用于高安全要求的生产环境，但对于个人博客这样的轻量级场景来说，已经是一个很好的平衡。

> 安全永远是一个权衡——在便利性和保护力度之间找到合适的平衡点
