# Awesome Tiandikuo 🚄✈️🚌

> 开源行程规划 / AI出行项目整理 — 为 [天地阔智能出行平台](https://www.tiandikuo.com) 提供技术借鉴

本仓库收集并分类整理了与天地阔产品愿景相关的开源项目，逐一分析每个项目的**可借鉴模块**和**不适用于天地阔的原因**，为技术整合提供决策依据。

## 天地阔产品愿景

天地阔是一个以移动端为核心载体的AI出行Agent：用户说一句话，系统理解需求→调用真实数据→输出可信可行方案→购票跳转→行程托管→偏差主动推送，让用户从"时刻盯着时间切换多个平台"变成"照做即可"。

**六层能力模型**：理解层→记忆层→推理层→执行层→兜底层→沟通层

---

## 目录

- [一、多模态出行编排引擎](#一多模态出行编排引擎决策层)
- [二、AI Agent框架与工作流](#二ai-agent框架与工作流编排层)
- [三、MCP工具服务器](#三mcp工具服务器数据接入层)
- [四、12306/票务数据源](#四12306票务数据源数据层)
- [五、地图与通勤计算](#五地图与通勤计算通勤层)
- [六、行程托管与状态机](#六行程托管与状态机托管层)
- [七、LLM应用与对话系统](#七llm应用与对话系统理解层)
- [八、前端出行UI组件](#八前端出行ui组件表现层)
- [九、出行数据与知识图谱](#九出行数据与知识图谱知识层)
- [十、跨端与移动端方案](#十跨端与移动端方案移动层)
- [附录：天地阔技术整合方案](#附录天地阔技术整合方案)

---

## 一、多模态出行编排引擎（决策层）

> 最接近天地阔"方案编排+多模态组合"定位的项目

### 1. [OpenTripPlanner](https://github.com/opentripplanner/OpenTripPlanner) ⭐ 2.1k
**语言**: Java | **定位**: 多模态行程规划引擎（GTFS+OSM）

| 维度 | 说明 |
|------|------|
| 核心能力 | 基于GTFS+OSM的多模态（公交/地铁/步行/自行车）A*路径规划，支持时间窗约束、 wheelchair可访问性 |
| 可借鉴模块 | ✅ A*多目标路径搜索算法（Pareto最优：时间/费用/换乘三维度同时优化）<br>✅ RAPTOR算法实现（公交时刻表高效查询）<br>✅ 时间窗硬约束处理（departureAfter/arrivalBefore）<br>✅ 多模态组合策略（walk→transit→walk）<br>✅ 方案去重与等价类合并 |
| 不适用原因 | ❌ Java技术栈，无法复用代码<br>❌ 面向城市内公交/地铁，不处理跨城铁路/航班<br>❌ 无自然语言理解层<br>❌ 无LLM集成<br>❌ 无中国交通数据适配 |
| 借鉴优先级 | **P0** — 天地阔的understand→validate→design链路需要Pareto多目标排序，当前sortPlans只按单一goal排序 |

### 2. [Valhalla](https://github.com/valhalla/valhalla) ⭐ 3.5k
**语言**: C++ | **定位**: 开源路由引擎（Mapzen/Mapbox）

| 维度 | 说明 |
|------|------|
| 核心能力 | 多模态路由（驾车/公交/步行/自行车）+ 时间距离矩阵 + isochrone等时线 |
| 可借鉴模块 | ✅ 多模态cost模型（toll/ferry/highway偏好权重）<br>✅ 时间距离矩阵API（多起点→多终点批量计算）<br>✅ isochrone等时线（"3小时内可达范围"可视化） |
| 不适用原因 | ❌ C++技术栈<br>❌ 无中国交通数据<br>❌ 无铁路/航班支持<br>❌ 无LLM集成 |
| 借鉴优先级 | **P2** — isochrone等时线可用于"推荐替代出发站"场景 |

### 3. [GraphHopper](https://github.com/graphhopper/graphhopper) ⭐ 4.8k
**语言**: Java | **定位**: 路由引擎（灵活权重）

| 维度 | 说明 |
|------|------|
| 核心能力 | A*/Dijkstra/CH路由 + 自定义权重模型 + 地图匹配 |
| 可借鉴模块 | ✅ 自定义权重模型（flexible weighting: user_priority函数）<br>✅ 地图匹配（GPS轨迹→路网snap，可用于定位偏差检测） |
| 不适用原因 | ❌ Java技术栈<br>❌ 无中国数据<br>❌ 无跨城能力 |
| 借鉴优先级 | **P2** — 地图匹配算法可借鉴于托管阶段的GPS定位snap-to-station |

### 4. [Yuntu (云图旅行)](https://github.com/3-19/Yuntu) ⭐ 待确认
**语言**: Python | **定位**: 确定性核心+发布门禁的旅行规划Agent

| 维度 | 说明 |
|------|------|
| 核心能力 | LLM两阶段生成：草稿→规则校验→发布门禁→定稿，确保方案零幻觉 |
| 可借鉴模块 | ✅ **发布门禁机制**（核心借鉴！草稿通过结构化规则校验才能发布，否则降级）<br>✅ 两阶段写入（draft→validate→publish，天地阔当前直接输出无校验）<br>✅ 确定性规则库（发车时间∈时刻表、价格∈[min,max]、换乘时间≥阈值）<br>✅ 反幻觉规则引擎（方案不通过门禁时标记unverified而非丢弃） |
| 不适用原因 | ❌ Python实现<br>❌ 无生产级部署<br>❌ 无跨城铁路数据 |
| 借鉴优先级 | **P0** — 天地阔最缺的能力：当前方案直接输出无门禁，需要draft→validate→publish链路 |

---

## 二、AI Agent框架与工作流（编排层）

> Agent工作流编排、Human-in-the-Loop、中断恢复

### 5. [SmartVoyage](https://github.com/AgiFlow/SmartVoyage) ⭐ 待确认
**语言**: TypeScript | **定位**: A2A协议+MCP+12306的多Agent旅行规划

| 维度 | 说明 |
|------|------|
| 核心能力 | A2A协议多Agent协作 + MCP工具调用 + 12306数据接入 + 两阶段规划 |
| 可借鉴模块 | ✅ MCP工具调用模式（12306作为MCP Server暴露query_trains/check_seats工具）<br>✅ 两阶段规划（fast-sketch→detail-fill）<br>✅ Agent间任务委托协议<br>✅ 12306数据在Agent间的传递格式 |
| 不适用原因 | ❌ A2A协议过重（天地阔单Agent+LangGraph已足够）<br>❌ 无生产级验证<br>❌ 无通勤/托管能力 |
| 借鉴优先级 | **P1** — MCP化12306接入模式可直接借鉴 |

### 6. [Ctrip-Style AI Travel Assistant](https://github.com/Ctrap-AI/Travel-Assistant) ⭐ 待确认
**语言**: Python/TypeScript | **定位**: Hub-and-Spoke架构+Human-in-the-Loop

| 维度 | 说明 |
|------|------|
| 核心能力 | LangGraph Hub-and-Spoke架构 + interrupt_before人工审批点 + 条件路由 |
| 可借鉴模块 | ✅ **Human-in-the-Loop**（核心借鉴！interrupt_before让用户审批/修改再继续）<br>✅ Hub-and-Spoke架构（中心调度→各领域子图）<br>✅ 条件路由（根据用户反馈动态选择下一个节点）<br>✅ 状态快照与恢复（interrupt后状态持久化，用户回来继续） |
| 不适用原因 | ❌ 天地阔已有LangGraph实现，不需要重写<br>❌ Python为主<br>❌ 无中国交通数据 |
| 借鉴优先级 | **P0** — Human-in-the-Loop是天地阔"增量修改"场景的关键缺失 |

### 7. [LangGraph](https://github.com/langchain-ai/langgraph) ⭐ 9k+
**语言**: Python/JS | **定位**: 有状态多Actor工作流框架

| 维度 | 说明 |
|------|------|
| 核心能力 | 有状态图执行 + checkpoint持久化 + interrupt_before/after + 人机协作 |
| 可借鉴模块 | ✅ interrupt_before（用户审批点）<br>✅ checkpoint持久化（状态快照+恢复）<br>✅ sub-graph组合（主图→子图委托）<br>✅ streaming事件协议 |
| 不适用原因 | ❌ 天地阔已使用LangGraph JS，但未用interrupt/checkpoint |
| 借鉴优先级 | **P0** — 启用LangGraph的interrupt和checkpoint能力（天地阔已有依赖但未使用） |

---

## 三、MCP工具服务器（数据接入层）

> MCP (Model Context Protocol) 将外部数据源标准化为LLM可调用的工具

### 8. [12306-mcp](https://github.com/Joooook/12306-mcp) ⭐ 待确认
**语言**: TypeScript | **定位**: 12306火车票MCP Server

| 维度 | 说明 |
|------|------|
| 核心能力 | 12306查询封装为MCP工具：query_trains/check_seats/query_price/query_station |
| 可借鉴模块 | ✅ **12306 MCP工具定义**（核心借鉴！天地阔当前12306通过后端API调用，MCP化可让LLM直接调用）<br>✅ 余票查询三态映射（available/limited/unavailable）<br>✅ 车次号→时刻表映射 |
| 不适用原因 | ❌ 12306无官方API，该实现基于逆向工程<br>❌ 查询频率受限<br>❌ 无购票能力 |
| 借鉴优先级 | **P1** — 12306 MCP化是天地阔推理层调用真实数据的关键升级 |

### 9. [trvl](https://github.com/MikkoParkkola/trvl) ⭐ 待确认
**语言**: Go | **定位**: 旅行规划MCP Server（1+66工具设计）

| 维度 | 说明 |
|------|------|
| 核心能力 | 1个MCP Server + 66个工具函数（覆盖航班/酒店/租车/火车/巴士/景点/货币/天气/签证/时区...） |
| 可借鉴模块 | ✅ **1+N工具设计模式**（1个MCP入口+N个细粒度工具，天地阔可定义travel_mcp含20+中国出行工具）<br>✅ 工具命名规范（search_flights/book_hotel/check_visa）<br>✅ 参数校验模式（必选/可选/默认值）<br>✅ 错误码统一 |
| 不适用原因 | ❌ Go语言<br>❌ 面向国际出行（Sabre/Amadeus GDS）<br>❌ 无中国数据 |
| 借鉴优先级 | **P1** — 工具设计模式和命名规范直接借鉴 |

### 10. [MCP Official Servers](https://github.com/modelcontextprotocol/servers) ⭐ 40k+
**语言**: TypeScript | **定位**: MCP官方工具服务器集合

| 维度 | 说明 |
|------|------|
| 核心能力 | 官方MCP Server集合：filesystem/sqlite/postgres/brave-search/github/google-maps等 |
| 可借鉴模块 | ✅ MCP Server标准实现模板<br>✅ postgres MCP（天地阔可直接用DB作为MCP数据源）<br>✅ google-maps MCP（地图查询标准化）<br>✅ SSE传输协议实现 |
| 不适用原因 | ❌ 无中国出行相关工具 |
| 借鉴优先级 | **P1** — 作为MCP实现参考 |

---

## 四、12306/票务数据源（数据层）

> 中国铁路票务数据获取方案

### 11. [12306-python](https://github.com/testerSunshine/12306) ⭐ 11k+
**语言**: Python | **定位**: 12306自动抢票（已归档）

| 维度 | 说明 |
|------|------|
| 核心能力 | 12306查询+下单+支付全流程自动化 |
| 可借鉴模块 | ✅ 12306请求加密机制（JS逆向）<br>✅ 余票查询API参数格式<br>✅ 车站代码→电报码映射 |
| 不适用原因 | ❌ Python实现<br>❌ 已归档(2020)<br>❌ 12306接口已多次变更<br>❌ 自动购票违反用户协议<br>❌ 法律风险极高 |
| 借鉴优先级 | **P3** — 仅参考API格式，绝不借鉴购票自动化 |

### 12. [py12306](https://github.com/pjialin/py12306) ⭐ 6k+
**语言**: Python | **定位**: 12306分布式抢票

| 维度 | 说明 |
|------|------|
| 核心能力 | 12306查询+多账号+分布式+Web管理界面 |
| 可借鉴模块 | ✅ 查询结果结构化格式<br>✅ Web管理界面设计（余票/车次/状态面板） |
| 不适用原因 | 同上 |
| 借鉴优先级 | **P3** |

### 13. [12306-booking](https://github.com/gzuii/12306-booking) ⭐ 待确认
**语言**: Vue | **定位**: 12306购票前端（学习项目）

| 维度 | 说明 |
|------|------|
| 核心能力 | 12306风格前端UI |
| 可借鉴模块 | ✅ 车次选择/座位选择/乘客信息UI交互设计 |
| 不适用原因 | ❌ 学习项目，无生产价值<br>❌ 无后端 |
| 借鉴优先级 | **P3** |

---

## 五、地图与通勤计算（通勤层）

> 门到门通勤计算、位置解析

### 14. [AMap (高德) JSAPI](https://github.com/amap/amap-jsapi) ⭐ 1k+
**语言**: TypeScript | **定位**: 高德地图JS API 2.0

| 维度 | 说明 |
|------|------|
| 核心能力 | 路径规划(驾车/公交/步行) + 地理编码 + 行政区查询 + 实时路况 |
| 可借鉴模块 | ✅ 天地阔已集成（amap.ts/amap-service.ts/enhanced-location-resolver.ts）<br>✅ 公交路径规划API（/v3/direction/transit/integrated）— 城市内，不支持跨城<br>✅ 驾车路径规划API — 已用于通勤计算 |
| 不适用原因 | 天地阔已集成，无需额外借鉴 |
| 借鉴优先级 | **已完成** |

### 15. [Leaflet](https://github.com/Leaflet/Leaflet) ⭐ 41k+
**语言**: JS | **定位**: 轻量级交互地图

| 维度 | 说明 |
|------|------|
| 核心能力 | 轻量级地图渲染 + 路线叠加 + 标记/弹窗 |
| 可借鉴模块 | ✅ 行程路线可视化（天地阔当前无地图可视化）<br>✅ 轻量级（38KB gzipped） |
| 不适用原因 | ❌ 天地阔使用高德JSAPI已有地图能力 |
| 借鉴优先级 | **P2** — 如果需要轻量级路线可视化可考虑 |

### 16. [Turf.js](https://github.com/Turfjs/turf) ⭐ 9k+
**语言**: JS | **定位**: 地理空间计算

| 维度 | 说明 |
|------|------|
| 核心能力 | 距离/面积/缓冲区/等时线等GIS计算 |
| 可借鉴模块 | ✅ distance计算（两点间Haversine距离，天地阔已有haversineKm）<br>✅ booleanPointInPolygon（判断用户位置是否在某区域内）<br>✅ destination计算（从某点沿某方向走N公里到达何处） |
| 不适用原因 | 天地阔已有haversineKm实现 |
| 借鉴优先级 | **P2** — booleanPointInPolygon可用于托管偏差检测 |

---

## 六、行程托管与状态机（托管层）

> 行程生命周期管理、偏差检测、替代方案搜索

### 17. [XState](https://github.com/statelyai/xstate) ⭐ 27k+
**语言**: TypeScript | **定位**: 有限状态机+状态图

| 维度 | 说明 |
|------|------|
| 核心能力 | 有限状态机定义+执行+可视化 + actor模型 + 持久化 |
| 可借鉴模块 | ✅ **custody_sessions状态机**（核心借鉴！planned→ticketed→in_transit→completed各状态转换规则）<br>✅ 状态转换守卫条件（canTransition: 只有ticketed才能进入in_transit）<br>✅ 状态持久化与恢复<br>✅ 可视化状态图（开发调试用） |
| 不适用原因 | ❌ 引入新依赖<br>❌ 天地阔custody_sessions已有简化状态管理 |
| 借鉴优先级 | **P1** — custody_sessions状态机化可大幅提升托管可靠性 |

### 18. [Temporal](https://github.com/temporalio/temporal) ⭐ 12k+
**语言**: Go/TS | **定位**: 长时工作流引擎

| 维度 | 说明 |
|------|------|
| 核心能力 | 长时工作流编排+持久化+重试+超时+信号+查询 |
| 可借鉴模块 | ✅ 长时工作流模式（行程托管可能持续数小时到数天）<br>✅ 信号机制（外部事件注入：列车晚点信号、用户位置更新信号）<br>✅ 自动重试+超时 |
| 不适用原因 | ❌ 过重（需独立Temporal Server）<br>❌ 运维成本高<br>❌ 天地阔2核8G服务器无法承担 |
| 借鉴优先级 | **P3** — 过重，但信号机制设计可借鉴 |

### 19. [BullMQ](https://github.com/taskforcesh/bullmq) ⭐ 6k+
**语言**: TypeScript | **定位**: Redis消息队列+任务调度

| 维度 | 说明 |
|------|------|
| 核心能力 | 分布式任务队列+延迟任务+重复任务+事件驱动 |
| 可借鉴模块 | ✅ 延迟任务（"出发前2小时推送提醒"=delayed job）<br>✅ 重复任务（"每5分钟检查一次列车状态"=repeatable job）<br>✅ 事件驱动（任务完成→触发下游） |
| 不适用原因 | ❌ 需要Redis（天地阔当前无Redis）<br>❌ 增加运维复杂度 |
| 借鉴优先级 | **P2** — 如果托管监控需要高频轮询，BullMQ是合理选择 |

---

## 七、LLM应用与对话系统（理解层）

> 多轮对话、偏好记忆、意图识别

### 20. [旅迹AI (Travel-Trace)](https://github.com/AusertDream/TravelPlanner) ⭐ 待确认
**语言**: Python | **定位**: 两阶段写入+DeepSeek+多平台API

| 维度 | 说明 |
|------|------|
| 核心能力 | DeepSeek驱动+12306/飞猪/途牛/高德API+两阶段写入+HTML路书生成 |
| 可借鉴模块 | ✅ **两阶段写入**（核心借鉴！LLM生成草稿→API数据校验→写入定稿）<br>✅ 多平台API聚合（12306+飞猪+途牛+高德同一查询并行调用）<br>✅ HTML路书生成（行程可视化文档）<br>✅ 发布门禁（API数据缺失时降级标记而非编造） |
| 不适用原因 | ❌ Python实现<br>❌ 无生产部署<br>❌ 无托管能力<br>❌ 无用户系统 |
| 借鉴优先级 | **P0** — 两阶段写入+发布门禁是天地阔最缺的核心能力 |

### 21. [LlamaIndex](https://github.com/run-llama/llama_index) ⭐ 40k+
**语言**: Python/TS | **定位**: RAG框架

| 维度 | 说明 |
|------|------|
| 核心能力 | 文档索引+语义检索+结构化数据查询 |
| 可借鉴模块 | ✅ agent_memories的语义检索（当前仅keyword匹配）<br>✅ 结构化数据查询（SQL+自然语言） |
| 不适用原因 | ❌ 天地阔记忆层尚未成熟，RAG不是当前瓶颈 |
| 借鉴优先级 | **P2** — 记忆层升级时考虑 |

### 22. [MemGPT/Letta](https://github.com/letta-ai/letta) ⭐ 13k+
**语言**: Python | **定位**: LLM长期记忆管理

| 维度 | 说明 |
|------|------|
| 核心能力 | 分层记忆（core/archival/recall）+ 自动记忆管理 + 上下文窗口管理 |
| 可借鉴模块 | ✅ **分层记忆**（核心借鉴！core=偏好/常用路线, archival=历史行程, recall=当前对话）<br>✅ 自动记忆管理（无需用户手动，系统自动归档/检索）<br>✅ 上下文窗口管理（长对话自动摘要压缩） |
| 不适用原因 | ❌ Python为主<br>❌ 天地阔记忆层优先级低于核心闭环 |
| 借鉴优先级 | **P1** — 分层记忆架构直接借鉴，用TS重新实现 |

### 23. [Vercel AI SDK](https://github.com/vercel/ai) ⭐ 12k+
**语言**: TypeScript | **定位**: AI应用开发SDK

| 维度 | 说明 |
|------|------|
| 核心能力 | useChat/useCompletion流式UI + tool calling + structured output + RAG |
| 可借鉴模块 | ✅ useChat流式对话（天地阔ChatInterface可部分参考）<br>✅ tool calling标准格式<br>✅ structured output（Zod schema→LLM输出保证结构化） |
| 不适用原因 | ❌ 天地阔已有自定义SSE实现，切换成本高<br>❌ 部分功能与LangGraph JS重复 |
| 借鉴优先级 | **P2** — structured output可用于understand层保证DemandStructure结构 |

---

## 八、前端出行UI组件（表现层）

> 行程卡片、时间线、方案对比

### 24. [Ant Design Pro](https://github.com/ant-design/ant-design-pro) ⭐ 36k+
**语言**: TypeScript | **定位**: 企业级前端方案

| 维度 | 说明 |
|------|------|
| 可借鉴模块 | ✅ 管理后台UI模板（天地阔admin页面可参考）<br>✅ ProTable/ProForm（行程管理/偏好设置表单） |
| 不适用原因 | ❌ Ant Design与天地阔shadcn/ui风格不一致 |
| 借鉴优先级 | **P3** |

### 25. [shadcn/ui](https://github.com/shadcn-ui/ui) ⭐ 80k+
**语言**: TypeScript | **定位**: 可组合UI组件

| 维度 | 说明 |
|------|------|
| 可借鉴模块 | ✅ 天地阔已使用shadcn/ui |
| 不适用原因 | 已集成 |
| 借鉴优先级 | **已完成** |

### 26. [React Timeline](https://github.com/cezarymp/react-timeline) ⭐ 待确认
**语言**: TypeScript | **定位**: React时间线组件

| 维度 | 说明 |
|------|------|
| 可借鉴模块 | ✅ 行程时间线可视化（出发→换乘→到达→办事→出发→...） |
| 不适用原因 | ❌ 天地阔已有自定义Timeline组件 |
| 借鉴优先级 | **P2** — 可参考交互设计 |

---

## 九、出行数据与知识图谱（知识层）

> 铁路网络、航班数据、城市关系

### 27. [china-train-gtfs](https://github.com/ViggoZ/china-train-gtfs) ⭐ 待确认
**语言**: Data | **定位**: 中国铁路GTFS格式数据

| 维度 | 说明 |
|------|------|
| 核心能力 | 中国铁路时刻表GTFS格式 |
| 可借鉴模块 | ✅ GTFS标准格式（天地阔direct_routes/graph_rail_connections可导出为GTFS，兼容OTP等引擎）<br>✅ 时刻表数据格式 |
| 不适用原因 | ❌ 数据可能过时<br>❌ GTFS不含余票/价格 |
| 借鉴优先级 | **P2** — 数据标准化参考 |

### 28. [china-citys-gtfs-bus](https://github.com/xyqfer/china-citys-gtfs-bus) ⭐ 待确认
**语言**: Data | **定位**: 中国城市公交GTFS数据

| 维度 | 说明 |
|------|------|
| 可借鉴模块 | ✅ 城市内公交数据（如果天地阔扩展到城市内规划） |
| 不适用原因 | ❌ 天地阔聚焦跨城出行 |
| 借鉴优先级 | **P3** |

### 29. [transit-feed](https://github.com/blinktaginc/transit-feed) ⭐ 待确认
**语言**: TypeScript | **定位**: GTFS数据生成工具

| 维度 | 说明 |
|------|------|
| 可借鉴模块 | ✅ 将天地阔direct_routes数据导出为GTFS格式 |
| 不适用原因 | ❌ 非核心需求 |
| 借鉴优先级 | **P3** |

---

## 十、跨端与移动端方案（移动层）

> APP壳、原生能力桥接、PWA

### 30. [Capacitor](https://github.com/ionic-team/capacitor) ⭐ 12k+
**语言**: TypeScript | **定位**: Web→原生桥接

| 维度 | 说明 |
|------|------|
| 核心能力 | Web App打包为原生APP + 原生能力插件（定位/推送/文件/后台） |
| 可借鉴模块 | ✅ **天地阔已采用**（capacitor.config.ts + native-bridge.ts + platform-detector.ts）<br>✅ Geolocation/PushNotifications/LocalNotifications插件 |
| 不适用原因 | 已集成 |
| 借鉴优先级 | **已完成** |

### 31. [Taro](https://github.com/NervJS/taro) ⭐ 15k+
**语言**: TypeScript | **定位**: 多端框架（微信/支付宝/H5/RN）

| 维度 | 说明 |
|------|------|
| 核心能力 | 一套代码编译到微信小程序/支付宝/H5/RN |
| 可借鉴模块 | ✅ 微信小程序端（天地阔如需小程序可考虑） |
| 不适用原因 | ❌ Taro不兼容Next.js（需重写）<br>❌ 天地阔选择Capacitor方案 |
| 借鉴优先级 | **P3** — 远期微信小程序时考虑 |

### 32. [Next.js PWA](https://github.com/shadowwalker/next-pwa) ⭐ 4k+
**语言**: TypeScript | **定位**: Next.js PWA支持

| 维度 | 说明 |
|------|------|
| 核心能力 | Service Worker + 离线缓存 + Web Push + 后台同步 |
| 可借鉴模块 | ✅ Service Worker（缓存行程数据供离线查看）<br>✅ Web Push（天地阔已集成BrowserNotificationManager）<br>✅ 后台同步（Background Sync API，表单离线提交） |
| 不适用原因 | ❌ next-pwa已不再维护（建议使用@ducanh2912/next-pwa） |
| 借鉴优先级 | **P1** — PWA可补足Web端离线能力 |

### 33. [Workbox](https://github.com/GoogleChrome/workbox) ⭐ 10k+
**语言**: JS | **定位**: Service Worker工具库

| 维度 | 说明 |
|------|------|
| 可借鉴模块 | ✅ 离线缓存策略（CacheFirst/NetworkFirst/StaleWhileRevalidate）<br>✅ 后台同步<br>✅ 推送事件处理 |
| 不适用原因 | ❌ 需要Service Worker配置 |
| 借鉴优先级 | **P2** — PWA底层依赖 |

---

## 附录：天地阔技术整合方案

### P0 整合项（~8人天）— 阻塞核心闭环

| 借鉴来源 | 整合内容 | 天地阔目标文件 | 验证标准 |
|---------|---------|--------------|---------|
| Yuntu #4 | 发布门禁机制 | `src/lib/travel/plan-gate.ts` (NEW) | 方案通过门禁才能输出，未通过标记unverified |
| 旅迹AI #20 | 两阶段写入 | `src/lib/agent/nodes/act.ts` | draft→API校验→publish，幻觉率=0 |
| Ctrip-Style #6 | Human-in-the-Loop | `src/lib/agent/graph.ts` | interrupt_before="design"让用户审批 |
| LangGraph #7 | checkpoint持久化 | `src/lib/agent/graph.ts` | 对话中断后可恢复 |
| OpenTripPlanner #1 | Pareto多目标排序 | `src/lib/travel/plan-ranker.ts` | 方案在time/price/comfort三维度Pareto最优 |

### P1 整合项（~7人天）— 体验升级

| 借鉴来源 | 整合内容 | 天地阔目标文件 | 验证标准 |
|---------|---------|--------------|---------|
| 12306-mcp #8 | 12306 MCP化 | `src/lib/mcp/train-mcp-server.ts` (NEW) | LLM可直接调用query_trains |
| trvl #9 | 1+N工具设计 | `src/lib/mcp/travel-mcp-server.ts` (NEW) | MCP入口+20个中国出行工具 |
| MemGPT #22 | 分层记忆 | `src/lib/agent/memory/` | core/archival/recall三层 |
| XState #17 | 托管状态机 | `src/lib/travel/custody-state-machine.ts` (NEW) | planned→ticketed→in_transit→completed |
| next-pwa #32 | PWA离线 | `next.config.js` + `public/sw.js` | 离线可查看已保存行程 |

### P2 整合项（~5人天）— 锦上添花

| 借鉴来源 | 整合内容 | 验证标准 |
|---------|---------|---------|
| Valhalla #2 | isochrone等时线 | "3小时内可达范围"可视化 |
| GraphHopper #3 | GPS地图匹配 | 定位snap-to-station |
| Turf.js #16 | 偏差区域检测 | 用户是否在"赶不上"区域 |
| BullMQ #19 | 托管高频轮询 | 每5分钟检查列车状态 |
| LlamaIndex #21 | 记忆语义检索 | 偏好/行程语义搜索 |

### P3 整合项 — 远期

| 借鉴来源 | 整合内容 |
|---------|---------|
| 12306-python #11 | 12306 API格式参考（绝不借鉴自动购票） |
| Taro #31 | 微信小程序端 |
| Ant Design Pro #24 | 管理后台UI |
| Temporal #18 | 长时工作流（过重，仅参考信号机制设计） |

---

## 统计

| 类别 | 项目数 | P0 | P1 | P2 | P3 | 已集成 |
|------|--------|-----|-----|-----|-----|--------|
| 多模态出行编排 | 4 | 2 | 0 | 1 | 0 | 0 |
| AI Agent框架 | 3 | 2 | 0 | 0 | 0 | 1 |
| MCP工具服务器 | 3 | 0 | 3 | 0 | 0 | 0 |
| 12306/票务数据 | 3 | 0 | 0 | 0 | 3 | 0 |
| 地图与通勤 | 3 | 0 | 0 | 2 | 0 | 1 |
| 行程托管与状态机 | 3 | 0 | 1 | 1 | 1 | 0 |
| LLM应用与对话 | 4 | 1 | 1 | 2 | 0 | 0 |
| 前端UI组件 | 3 | 0 | 0 | 1 | 1 | 1 |
| 出行数据与图谱 | 3 | 0 | 0 | 1 | 2 | 0 |
| 跨端与移动端 | 4 | 0 | 1 | 1 | 1 | 1 |
| **合计** | **33** | **5** | **6** | **9** | **8** | **4** |

---

## 贡献

本仓库由天地阔项目维护。如需补充新的开源项目，请提交Issue或PR。

## 许可

本仓库内容仅供技术调研参考，各开源项目版权归原作者所有。
