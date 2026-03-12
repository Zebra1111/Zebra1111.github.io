## 项目介绍

黄金矿工是一个经典的 4399 小游戏，这次我用纯前端技术把它复刻了出来，作为我的 GitHub 主页。

### 技术栈

- **HTML5 Canvas** — 游戏渲染
- **原生 JavaScript** — 游戏逻辑
- **CSS3** — UI 界面

### 核心逻辑

游戏主要分为以下几个模块：

1. **钩爪摆动** — 使用三角函数控制角度
2. **碰撞检测** — 计算钩爪与物品的距离
3. **状态机** — swinging / extending / retracting

```javascript
// 钩爪位置计算
miner.hookX = miner.x + Math.sin(miner.angle) * miner.ropeLen;
miner.hookY = miner.y + Math.cos(miner.angle) * miner.ropeLen;
```

### 粒子效果

抓到物品时会触发金色粒子爆炸效果，增加游戏的视觉反馈。

> 用简单的数学就能创造有趣的游戏体验
