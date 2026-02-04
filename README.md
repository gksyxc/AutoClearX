# AutoClearX

AutoClearX is a Minecraft Java **Paper** server plugin designed to automatically manage excessive entities and dropped items using **performance-based thresholds** rather than fixed limits.

AutoClearX 是一个用于 Minecraft Java **Paper 服务端**的自动清理插件，核心目标是基于**服务器真实性能**而不是固定经验值来管理实体与掉落物。

---

## Core Idea / 核心思想

AutoClearX does **not** blindly clear entities on a fixed schedule.
Instead, it first benchmarks the server to understand how entity count affects MSPT, then derives safe cleanup thresholds from real measurements.

AutoClearX 并不会定时全服扫地。
插件会先对服务器进行压力测试，测量实体数量对 MSPT 的影响，再根据实测结果计算安全的清理阈值。

---

## Main Features / 主要功能

### 1. Automatic Benchmarking / 自动性能测试

- Runs automatically after server startup (with a delay).
- Uses a dedicated flat test world to measure:
  - Single-chunk concentrated entity pressure
  - Multi-chunk distributed entity pressure
- Calculates slopes of `entities → MSPT` and stores results persistently.

- 服务器启动后自动延迟执行跑分测试  
- 在独立的测试世界中测试：
  - 单区块集中实体压力
  - 多区块分散实体压力
- 计算“实体数量 → MSPT”的斜率并持久化保存

---

### 2. Performance-Based Thresholds / 基于性能的阈值

- Cleanup thresholds are derived from benchmark results.
- Thresholds represent the estimated entity count that would push MSPT toward ~50ms.
- Uses:
  - Multi-chunk slope to limit global entity count
  - Single-chunk slope to limit per-chunk entity density

- 清理阈值完全来源于跑分结果  
- 阈值表示“接近 50ms MSPT 时的实体规模”  
- 使用：
  - 多区块斜率约束全服总量
  - 单区块斜率约束热点区块

---

### 3. Incremental Monitoring / 增量式统计

- Continuously scans **loaded chunks only**
- Scans **10 chunk per second**, prioritizing player surroundings
- Dynamically updates statistics as chunks load/unload
- Avoids full-world scans

- 仅统计**已加载区块**
- 每秒只扫描 **10 个区块**
- 优先扫描玩家附近区块
- 区块卸载后自动移出统计列表，避免全服扫描

---

### 4. Countdown-Based Cleanup / 倒计时清理机制

- When thresholds are exceeded:
  - Starts a 60-second countdown
  - Notifies players near the target chunk
- At 10 seconds remaining:
  - Re-checks conditions
  - Cancels cleanup if load has dropped
- Cleanup only affects **one target chunk**

- 达到阈值后启动 60 秒倒计时
- 仅通知目标区块附近玩家
- 倒计时剩余 10 秒时复查
- 低于阈值则取消
- 清理仅作用于**单个区块**

---

### 5. Safe Cleanup Rules / 安全清理规则

- Removes:
  - Dropped items
  - Unnamed living entities
- Honors whitelist rules from config
- Does not touch blocks, containers, or named entities

- 清理对象：
  - 掉落物
  - 未命名生物
- 支持配置白名单
- 不修改方块、不处理容器、不清理命名实体

---

### 6. Public Trash Container / 公共垃圾桶

- All cleaned items go into a shared inventory
- Inventory size: one double chest (54 slots)
- Uses rolling overwrite:
  - Items are placed sequentially
  - Old items are overwritten when full

- 清理产生的物品进入公共垃圾桶
- 大小固定为一个大箱子（54 格）
- 采用滚动覆盖策略：
  - 按顺序放入
  - 满了直接覆盖旧内容

---

## Commands / 指令

### Sweep Commands / 扫地指令

- `/sweep status`
  - Shows current statistics, thresholds, countdown state
- `/sweep now`
  - Immediately cleans the current target chunk

- `/sweep status`
  - 查看当前统计、阈值、倒计时状态
- `/sweep now`
  - 立即清理当前目标区块

---

### Benchmark Commands / 跑分指令

- `/sweepbench status`
- `/sweepbench run`
- `/sweepbench clear`

用于查看、手动触发或清理跑分测试。

---

## Configuration / 配置

Only whitelist-related options are configurable.

仅支持白名单配置，其余逻辑全部自动化。

- Item material whitelist
- Entity type whitelist

---

## Design Philosophy / 设计理念

- Prefer **self-adaptive limits** over hard-coded values
- Avoid aggressive or global cleanup
- Accept brief local slowdown to achieve long-term stability
- Optimize for real servers, not synthetic benchmarks

- 优先使用**自适应阈值**
- 避免激进或全服扫地
- 接受短暂局部影响以换取长期稳定
- 面向真实服务器环境而非理论极限

---

## Notes / 注意事项

- Extremely powerful servers may reach entity cramming or client limits before MSPT degrades.
  The benchmark module includes hard caps to avoid meaningless test ranges.

- 在性能极强的服务器上，可能在 MSPT 变化前先触及挤压或客户端极限  
  跑分模块已包含上限以避免进入无意义测试区间


