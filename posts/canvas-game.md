## 黄金矿工 Canvas 复刻：技术实现复盘

这个项目不是“静态主页 + 动效”，而是一个完整的单文件游戏：在首页直接运行、可交互、可计分、可失败、可重开，并且把个人链接融入到游戏奖励系统中。

---

## 技术框架

### 1) 渲染层：HTML5 Canvas + requestAnimationFrame

- 使用 `canvas.getContext('2d')` 统一负责场景绘制。
- 主循环 `gameLoop()` 每帧按固定顺序执行：
	1. 绘制背景/星空/地面纹理
	2. 绘制物品、粒子、矿工、绳索与钩爪
	3. 更新矿工状态、粒子状态、关卡重生逻辑
- 用 `requestAnimationFrame(gameLoop)` 保证刷新节奏与浏览器渲染同步。

### 2) 逻辑层：原生 JavaScript 状态机

- 采用集中式 `miner` 状态对象管理钩爪行为。
- 三态状态机：
	- `swinging`：左右摆动等待发射
	- `extending`：绳索伸长并检测碰撞
	- `retracting`：回收并结算得分/失败
- 所有核心逻辑都在 `updateMiner()` 中闭环，行为可预测、易维护。

### 3) 资源与显示层：图片预加载 + 高 DPI 清晰度优化

- 金、银、石头等素材在启动时预加载到 `imageCache`。
- 兼容高分屏：根据 `devicePixelRatio` 调整 `canvas.width/height`，并同步 CSS 尺寸。
- 开启 `ctx.imageSmoothingEnabled = true` 与 `imageSmoothingQuality = 'high'`，减少图标边缘锯齿。

---

## 实现亮点

### 亮点一：把“个人主页导航”做成游戏奖励

可抓取对象不仅是资源，还包含外链与站内页：GitHub、博客、项目页、LeetCode。

- 抓到链接类物品时弹窗确认（打开页面 / 继续挖矿）
- 也支持鼠标直接点击物品触发
- 游戏与内容入口合并，减少传统导航栏跳转的割裂感

### 亮点二：可达性约束，避免“刷出抓不到的物品”

物品生成时加入 `isReachable(x, y, size)`：

- 角度必须落在摆动扇形内（`maxSwingAngle`）
- 距离必须小于钩爪最大可达长度（考虑物体半径留白）

同时 `maxRopeLen` 依据视口动态计算，窗口变化后仍能覆盖有效区域。

### 亮点三：反馈链路完整

- 得分实时刷新（顶部 HUD）
- 抓取成功触发粒子
- 抓到炸弹进入 Game Over 弹层
- 全部物品清空后自动重生下一轮

这套反馈让“发射 → 命中 → 结算”每一步都有视觉和数值响应。

---

## 核心游戏逻辑拆解

### 1) 钩爪运动与摆角

摆动阶段通过角速度与方向标记控制左右往返：

```javascript
miner.angle += miner.angleSpeed * miner.angleDir;
if (miner.angle > miner.maxSwingAngle) miner.angleDir = -1;
if (miner.angle < -miner.maxSwingAngle) miner.angleDir = 1;
```

钩爪端点坐标用三角函数计算：

```javascript
miner.hookX = miner.x + Math.sin(miner.angle) * miner.ropeLen;
miner.hookY = miner.y + Math.cos(miner.angle) * miner.ropeLen;
```

### 2) 碰撞检测与抓取

在 `extending` 阶段遍历存活物体，按圆形近似做距离判定：

```javascript
const dist = Math.hypot(miner.hookX - item.x, miner.hookY - item.y);
if (dist < item.size / 2 + 15) {
	miner.grabbed = item;
	item.alive = false;
	miner.state = 'retracting';
}
```

抓到后回收速度会按物品大小做衰减，大物体回收更慢，保留经典黄金矿工手感。

### 3) 结算机制

回收到起点后分三类处理：

- `bomb`：触发失败弹窗 + 红色爆炸粒子
- 普通资源：加分 + 金色粒子
- 链接资源：加分后再弹出跳转确认

### 4) 场景更新与循环再生

- 粒子系统每帧衰减透明度并受重力影响
- 当所有物品 `alive=false` 时，自动触发下一轮生成

这保证了游戏是“可持续运行”的，而不是一次性演示。

---

## 总结

这个项目的关键价值不只是复刻玩法，而是把 **交互式小游戏、个人信息入口、工程化渲染细节** 融到同一页面里：

- 用最少依赖实现完整状态机游戏
- 用可达区域约束提升可玩性
- 用高 DPI + 抗锯齿提升视觉品质

一句话：这是一个“可玩的个人主页”，而不是“会动的简历页”。
