# AGENTS.md · 环游集（china-travel-atlas）

## 这个项目是什么

单文件网页：`index.html`（**零构建、零 npm 依赖、零外部 JS/CSS 库**）。地球 → 国家 / 地区 → 城市 → 景点 四级下钻 + 我的清单；每个景点聚合「介绍 + 真实照片 / 卫星俯瞰 + 站内可看的攻略帖 + 可复制 / 下载的攻略文本」。

改文档看 `README.md`（含每一版的改动与验证方式）；改代码只改 `index.html`；`tools/` 下是零依赖的校验与探针脚本。

## 长期约定（必须遵守）

1. **每次迭代都增加一些城市和景点。** —— 用户的明确要求。任何一次功能改动，都要顺带扩充数据：新增城市写**新的数据块**（如 `CITIES_CN_V7` / `CITIES_CN_V10`）+ 一段 `concat` / `WORLD.push` 合并，**不动老数据**；`desc` ≥ 20 字、`tip` 不能空，`heat` 在 60–100。**v10 起的下限：每城 ≥15 个景点；省级行政区层面，省 / 自治区每处 ≥3 城，直辖市与特别行政区 ≥1 城。** v8 只加了交通层、v9 只加了界面与检索，都明确标注为一次性例外；**v10 一次补齐：95 座老城各 +5 个点位（补到 15）+ 5 座新城市（天水 / 酒泉 / 嘉峪关 / 赤峰 / 克拉玛依，各 15 个点位）+ 铁路网 40 → 71 条干线**；**v11 继续加 10 城（平凉 / 庆阳 / 汉中 / 安康 / 上饶 / 许昌 / 赣州 / 郴州 / 宜宾 / 泸州，各 15 个点位），铁路补到 73 条并把 10 座新城接进线里**。**v12.1 是用户点名的本地模型链路维护，只升级 WebLLM / Qwen 档位与下载回退，不扩城市 / 景点（一次性例外）**。**v13 是用户点名的「中国旅行工作台」体验升级：只补景点实用信息字段、日期闭馆、换城减负、预算住宿，不扩城市 / 景点（一次性例外）**。下一轮按同样下限继续加城 / 加点，别再拿「只加功能」当理由。
1.1 **大批量数据走「手稿 + 生成器」**：v7 的点位与榜单写在 `tools/data/cn-v7-*.mjs`，跑 `node tools/gen-cn-v7.mjs` 写 `V7-DATA:BEGIN…END`；v10 的补点 / 新城写在 `tools/data/cn-v10-*.mjs`（一行一个点位：`id|名称|类别|纬度|经度|热度|票价|介绍|建议`），跑 `node tools/gen-cn-v10.mjs` 写 `V10-DATA:BEGIN…END`（幂等；生成器会拦「字段缺失 / 坐标不在中国版图内 / 票价缺失或格式不对 / 老城补点少于 5 / 新城少于 15 / id 与现有点位撞车 / 榜单缺项」）。**v11 沿用同一格式**：`tools/data/cn-v11-*.mjs` → `node tools/gen-cn-v11.mjs` → `V11-DATA:BEGIN…END`（校验每城 15 个点位、9 列格式 / 类别白名单 / 中国版图 / 热度 60–100 / 票价格式 / 介绍 ≥20 字 / id 唯一 / 榜单四项）。**不要手改标记区**——改数据改手稿再重跑；`index.html` 本身依旧是零依赖、可直接双击打开的单文件。
2. **页脚 / 地球按钮上的规模数字是运行时生成的**（`stampCounts()`）；扩数据**不用**改文案，但 `README.md` 的数字与 `tools/verify-*.mjs`、`tools/check.mjs` 里的规模断言要**一起上调**（不允许「数据涨了、断言不动」）。当前规模：**22 国 / 174 城 / 2060 景点**（中国 115 城 / 1765 景点，34 个省级行政区全覆盖）。
3. **收工前必须全绿**：
   - `node tools/check.mjs`（语法 / DOM 选择器 / 数据完整性 / 投影复算 / 各版本功能点，含 [13] v6 三项、[14] v7 四项、[15] v8 两项、[16] v9 两项、[17] v10 五项、[18] v11 四项、[19] v12 十项与 [20] v13 八项）
   - `node tools/verify-v3.mjs`、`node tools/verify-v4.mjs`、`node tools/verify-v6.mjs`、`node tools/verify-v7.mjs`、`node tools/verify-v8.mjs`、`node tools/verify-v9.mjs`、`node tools/verify-v11.mjs`、`node tools/verify-v12.mjs`、`node tools/verify-v13.mjs`（真实浏览器 + CDP 的行为断言；v7 的天数 / 分组 / AI 面板场景跑 `node tools/probe-v2-all.mjs v7Cities v7PlanDays v7PlanFix v7AiPanel v6Plan v6PlanAuto v6PlanJump`，v8 的去重 / 交通网 / 城际段场景跑 `node tools/probe-v2-all.mjs v8Dedupe v8NetMap v8NetZoom v8CityNet v8PlanLegs` —— 其中 `v8NetMap` 带 `dsf=2`，专抓高分屏下画布被拉伸的问题；v9 的附录 / 定位层场景跑 `node tools/probe-v2-all.mjs v9Home v9Appendix v9At v9GeoMock v9GeoOffline`；v10 的行程可编辑 / 票价场景跑 `node tools/probe-v2-all.mjs v10PlanEdit v10Fee`；v11 的 110 城 / 距离分天 / 曲线铁路 / 真实底图场景跑 `node tools/probe-v2-all.mjs v11Cities v11Distance v11NetCurve v11MapBase`；v12 的 115 城 / 路上时间 / 食巷 / 12306 面板场景跑 `node tools/probe-v2-all.mjs v12Plan v12RailUI`；要验证真代理出余票，先 `node tools/12306-proxy.mjs`，再跑 `v12RailLive`；v13 的日期闭馆 / 换城减负 / 预算住宿场景跑 `node tools/probe-v2-all.mjs v13Plan v13Close`）
   - `node tools/probe-v2-all.mjs`（60 个默认场景 + 可选 `v12RailLive` 的全量截图 + 报告，写 `tools/_shots/_v2_summary.json`；`v12RailLive` 需要先开本机代理，未开代理时默认跳过；只跑部分场景时结果并回 summary）
4. **零依赖**：不要引入 npm 包 / **外链 `<script src>`** / 构建步骤；新增能力要么写进 `index.html`，要么写成 `tools/` 下的纯 Node 脚本。唯一例外：**站内 AI 的本地模型引擎**（WebLLM）允许在用户**点击「启动本地模型」时**用动态 `import()` 取远端脚本 —— 首屏（含所有校验脚本）不得发起任何 CDN / 大模型请求，`tools/verify-v7.mjs` 会断言这一点。
5. **合规底线**（不要为了「效果」破例）：
   - 图集只放**真实影像 / 摄影**，**绝不使用 AI 生成图**，也不用来路不明的图凑数；
   - 获奖作品、平台内容**只给外链**（不转载、不热链），运行时不抓取第三方页面正文 / 图片；
   - **「必吃榜 / 必去榜 / 热门路线」只放事实性名称**（菜名 / 景点名 / 路线节点顺序）+ 自己写的说明 + 来源链接，**不摘抄平台正文、不放平台截图**；界面上必须写清「不是官方排名」与「检索日期」（`LIST_UP`，改榜单数据时同步改）；
   - **行程助手的「AI」由用户自己开通道**：本站不代理、不代问、不存对话。①**自带 Key 直连**（OpenAI 兼容：DeepSeek / 通义 / 智谱 / Moonshot / OpenAI / 本机 Ollama / 自定义），Key 只落本机 `localStorage`（`cta.ai.v1`），请求从浏览器直接发给服务商，提示里必须写明「数据会发给谁」；②**本机小模型**（WebLLM 0.2.86 + Qwen3.5，用户点「启动本地模型」时才动态 `import()` 并下载权重；默认国内镜像、可切官方源，显存不够如实报错）。**没 Key、没开本地模型时，行程照旧全在本地算**；要更细的信息仍可给「口令 + 站外跳转」，不抓取第三方正文。
   - 用户数据（收藏、图库 Key、自己的照片、行程条件）只落本机 `localStorage`，**不上传**；
   - 中国地图部分明确标注「国界与南海断续线为演示用示意界线，正式上线须替换为自然资源部标准地图」。

## 已知环境限制（别浪费时间重复踩）

- `commons.wikimedia.org` / `upload.wikimedia.org` / `api.wikimedia.org` / `api.openverse.org`：**本机不可达**（超时 / 连接重置）。
- `www.pexels.com` 搜索页：**Cloudflare 人机验证**（CDP 真浏览器也一样）；`api.pexels.com` / `api.unsplash.com` 需 Key（`images.pexels.com` / `images.unsplash.com` CDN 本身 200 + `ACAO:*`，可直链）。
- **12306（v12 实测）**：`kyfw.12306.cn/otn/leftTicket/query` 直接 fetch 返回 `text/html` 风控页且 **无 ACAO**；先 `GET /otn/leftTicket/init` 拿 `JSESSIONID` Cookie，再带 `Referer` / `X-Requested-With` 请求，才返回 `application/json`（实测 2026-10 北京南→上海虹桥 55 趟）。站名表 `otn/resources/js/framework/station_name.js` 可解析「中文名 → telecode」。因此浏览器必须经 `tools/12306-proxy.mjs`（只监听 127.0.0.1），**不要**尝试在页面里直连 12306；代理也不登录、不购票、不保存账号。
- **v6 新测**（`node tools/probe-images.mjs`，结果落 `tools/_shots/_images.txt`）：Unsplash / Pexels / Pixabay 的**搜索页**分别是 401 / 403 / 403（HTTP 客户端也拿不到），`cdn.pixabay.com` 403 —— 所以「按关键词批量取免费图库照片」在本机做不到，内置真图只能靠**逐张确认过的精选 + 卫星实景**；**高德卫星 z16/z17 与 Esri 影像瓦片 200 + `ACAO:*`**（能拼图、能导 PNG）；榜单来源侧：Bing 200、携程攻略 200、去哪儿 200、知乎 200、小红书 200、大众点评 200、马蜂窝 202、百度百科 403。
- **v9 新测**（`node tools/probe-geocode.mjs`，结果落 `tools/_shots/_geocode.txt`）：`nominatim.openstreetmap.org` **ECONNRESET（本机被拦，几条查询全挂）**；`photon.komoot.io` **200 + `ACAO:*`**（「杭州 西湖区 文三路」→ 文三路 30.2790/120.1227 ·「天水」→ 34.5671/105.8949 ·「上海虹桥站」→ 31.1961/121.3161）。**Photon 不支持 `lang=zh`**（三查三 400），别再给它加这个参数。所以页面里 Photon 主、Nominatim 备。
- Bing 图片搜索可达，但返回的是新浪 / 凤凰 / 699pic 等**版权或商业图库**的图 —— 不能用。
- 终端回显不可靠：长时间任务用 `Start-Process -WindowStyle Hidden` + 重定向到文件，再用 `read_files` 读。
- 别用 `search_codebase`，用 `Select-String` 把结果写成文件再读（`-Encoding utf8`）。

## 代码约定（`index.html`）

- 数据字段：城市 `id / name / lat / lng / heat / tag / tags / pois[]`；景点 `id / name / cat / lat / lng / heat / desc / tip`，v10 起新增 `fee`（`免费` / `¥60` / `¥418 起`，收费景区必须写参考价）；国家 `id / name / heat / flag / region / tag / intro / cities[]`。
- `id` 全局唯一、URL 安全（会进 `#/c/国家/城市/景点`）；POI id 一律小写字母 / 下划线。
- 坐标系分开：中国段按高德 GCJ-02 近似值，海外段 WGS-84 近似值。
- 新国家的国旗在 `FLAGS` 里用色带 / 圆点 / 星拼（Windows 不渲染旗帜 emoji）。
- 地图标签（`placeMarkers()`）：给 **13 个候选方位**（8 个贴光点方位 + 4 个「再挪一格」+ 1 个兜底把名字居中压在光点上），标签宽高必须**实测**（`sizeOf()`），不要用 `name.length * 13` 估算；顶栏有「🏷 标签：自动 / 全部」强制全显（记忆在 `cta.lbl.mode`）。
- 国家层镜头（`countryFocus()`）：**按城市包围盒自适配**缩放（`MERC_Y` 量经度 / 纬度跨度 → 视口尺寸反推 zoom，四周留 12%），不要退回「按城市离质心多远查表」——那会让「两城相隔很远」的国家（希腊 雅典↔圣托里尼、葡萄牙 里斯本↔波尔图）把城市全都甩出画面，只剩空海。
- 中文文案风格：直白、给理由（「为什么这么改」写在注释里），避免夸张营销词。
- **内置图片不要「假装有图」**：能拿到的真图只有两类 —— ① 逐张确认过的 `PHOTO_PICKS`（Pexels CDN，实测 200 + `ACAO:*`）；② **卫星实景瓦片**（高德国内 / Esri 海外，实测同样 200 + `ACAO:*`，所以画布不被污染、`toDataURL` 导得出来）。瓦片一律用 `img.crossOrigin = 'anonymous'` 取，**没有 ACAO 时自动回退并标记 `satState.tainted`**，导出按钮要如实说明「右键另存」，不要假装导出成功。
- 卫星实景（v6）：三档 `SAT_ZOOMS`（z17 / z16 / z14）共用 `satPaint({cv, lat, lng, srcKey, z, cols, rows, mark})`；同城图墙 `drawCityWall()` 一律 z15 + 2×2，每城最多 8 张，避免一次打出太多瓦片请求。
- 「我的照片」（v6）：入口三条（单条直链 / 多行批量 / 拖拽文件），压缩走 `compressImage()`（≤1600px JPEG），**只落本机 localStorage**；`🩺 图片体检` 用真加载（`new Image()`）逐条报可用性，不要用 `fetch` 代替（跨域会把正常图也判成失败）。
- 行程助手（v7 / v8 / v10）：`PLAN_KEY = 'cta.plan.v1'` 存条件；**天数一颗不少**——`plPickCities()` 权重 = `heat + pois×3 + 命中偏好×6`，最大余数法把 `days` 分完（每城 ≥1 天），**没有单城 3 天 / 5 天封顶**；用户在「按城市微调」里固定的天数（`cfg.fix`）优先，固定值总和超过滑块时自动抬高总天数并提示。**v8 起 `plBuild()` 是「消费式取用」**：白天点位 `dayQ`、夜景点 `nightQ`、两餐 `eatQ` 各自一条队列，`splice` 取走就不放回，**一趟行程里同一景点最多出现一次**（`__CTA_PLAN_LAST.dups` 必须为 0）；点位排完的城市剩下的天数写成「放慢节奏 / 自由活动」并给出建议天数；换城日 `plLeg()` / `plRailBetween()` 先在 `CN_RAIL_LINES` 里找直达干线，找不到再找**一次换乘**（两条线共用同一个城市站点），都没有才按距离给建议（口径「以 12306 为准」）。**v10 起行程可编辑**：每个地点旁 `⇄` 换一个 / `✕` 不去，排除项存 `planCfg.drop`（`cityId/poiId`），`#plUndo` 一键恢复；`#plCities` 顶部固定一条「已选目的地」置顶条，**任何时候都能取消已选城市 / 清除搜索筛选**——不许再出现「锁死在南京、改不了」的观感。目的地清单按**省级行政区**分组（`CN_PROV` / `CN_PROV_KIND`）+ 搜索框，115 城一屏可查。**v11 起按距离排**：城际用「最近邻 + 2-opt」消折返（用户选的第一座城仍是起点），城内先最近邻 + 2-opt、再从「热度 / 偏好排序」里按最远点选中心 + 最近点配对 + 按距离补满，把白天点位分成每天一簇（`anchorPool` 必须按 `plScoreOf()` 排，不能拿城内路线顺序当锚点）；夜景全局挂到「最后一个白天点离它最近」的那天（不要按天号先到先得，否则都江堰 / 青城山会被硬塞一个市中心夜景）。**v12 起按「路上时间」做分簇局部改良**：`plTravelMin()` 把 km 换算成分钟（步行 4.5 / 公交地铁 18 / 打车 30 km/h + 候车），`plRouteMinutes()` 评估一天路线，`plBucketPolish()` 在两天之间交换 / 移动点位，只要两天路上时间之和变小就接受；每天同时报点位跨度与 `transitMin`，超过节奏建议（轻松 60 / 标准 90 / 紧凑 120 分钟）或单段 >35 km 就在卡片上提醒。**v12 食巷**：`bites` 是独立类别，`biteQ` 一天最多一个、整趟不重复，全局挂到「最后一个白天点离它最近」的那天；没有手写食巷的城市从已有 `food` / `street` 里补一个（`p.bite = true`），保证每城都有入口。**v12 12306**：`plLegInfo()` 给每段城际结构化站名，`plRailQuery()` 经本机代理查实时余票；没启动代理时只提示未连接，绝不假装同步。**只做中国**（`plCandidates()` 取 `citiesOf('cn')`），预算 / 里程标注为「演示级估算」。改算法后跑 `node tools/verify-v7.mjs` + `node tools/verify-v8.mjs` + `node tools/probe-v2-all.mjs v7Cities v7PlanDays v7PlanFix v8Dedupe v8PlanLegs v10PlanEdit v10Fee v11Cities v11Distance`（断言 cityChips ≥111、10 天 → 10 张卡片、成都 7 天 `dups=0`、`legs` 是真实干线名、点北京后南京仍在、置顶条能取消南京、`drop` 1 → 2、remoteHits 为空）。
- 交通网（v8 / v10 / v12）：`CN_RAIL_LINES`（**73 条干线**，`stops` 是 `[城市 id 或 null, 站名, lat, lng]`）+ `CN_CITY_NET`（**115 城** `st / ln / mt / bs / tip`，其中 `ln` 由生成器从干线站点推导）+ `CN_NET_NOTE`（口径）。手稿 `tools/data/cn-v8-*.mjs` → `node tools/gen-cn-v8.mjs` → 写 `V8-NET:BEGIN…END` 标记区，**不要手改标记区**。`Net` 模块只画线不改地图状态：`#net` 必须 `pointer-events:none`（否则拖拽被截断），缩放 < 7 不写站名、≥ 9.8 调淡，状态存 `cta.net.v1`，探针钩子 `window.__CTA_NET`（`on / set / lines / painted / labels / cities / note / box`）。**线形是站间平滑示意曲线**（`smoothPath()`，Catmull-Rom → 三次贝塞尔，只为视觉顺滑），不许写成「真实线路中心线」；站点小圆点仍按原始坐标画，班次一律以 12306 为准，公交只写枢纽名、不编路线号。v10 补的是**普速主干 + 高铁通道**：陇海 / 包西 / 西延 / 京通 / 奎北 / 郑渝 / 汉十 / 兰渝 / 成昆复线 / 贵南 / 赣深 / 昌赣 / 合安九 / 杭黄 / 商合杭 / 宁杭 / 沪苏湖 / 福厦 / 郑太 / 石济 / 津秦 / 京唐 / 张吉怀 / 黔张常 / 拉日 / 格库 / 宝成 / 襄渝 / 焦柳 / 滨洲 / 滨绥，并把天水 / 酒泉 / 嘉峪关 / 赤峰 / 克拉玛依 / 承德 / 延安接进线里；`gen-cn-v8.mjs` 的省份表同时读 `CN_PROV_V10` / `CN_PROV_V11` / `CN_PROV_V12`，否则新城市会被判成「非中国城市」。v11 补了宝中铁路（平凉）与绵泸高铁（泸州），并把上饶 / 许昌 / 汉中 / 安康 / 庆阳 / 赣州 / 郴州 / 宜宾改成真实城市站点。v12 把淄博接进胶济客专、佛山西接进贵广高铁、柳州接进湘桂 / 焦柳、漳州接进福厦高铁、台州接进杭深铁路，`CN_CITY_NET` 同步到 115 城。
- 交通网画布（v8 修复，别再踩）：`#net` 是 `<canvas>` = **替换元素**，只写 `position:absolute;inset:0` 时 `width:auto` 会取**位图宽度**当 CSS 宽度 —— `devicePixelRatio > 1`（Windows 缩放 150% / 200%、浏览器缩放 ≥110%）时画布被撑成 `vw×dpr`，整层铁路放大 2 倍飘到海上。所以：① `#net` 必须显式 `width:100%;height:100%`（`check.mjs [15]` 会拦）；② 位图尺寸按 `cv.clientWidth / clientHeight × dpr` 开（`fitCanvas()`），不要用 `onDraw` 回调里的 `vw/vh`；③ 几何取 `MapView.zoom / center` 的当前值，画布盒与 `vw/vh` 差 >1px 时跳过该帧并下一帧 `MapView.resize()` 自愈。headless 探针默认 `dpr=1` 抓不到这类问题 —— 图层场景用 `probe-dom.mjs` 第 8 个参数 / 场景数组第 8 位设 `dsf=2`（`v8NetMap` 就是这么跑的），再由 `report.planner.net.align` 断言「`stretch=0` + 京沪线上有像素 + 公海零像素」。
- 附录浮层（v9）：页脚只留 `.foot-bar` 一行；原来那十几行说明整块放进 `<section class="apx hidden" id="appendix">`，`.apx{position:fixed}` —— **不许改回文档流里的折叠面板**（那会把 `#stage` 压矮；`verify-v9` 用「打开前后 `stageH` 差 ≤2px」拦）。关闭三条路：`#apxClose` / `Esc` / 点浮层外空白；按钮的 `aria-expanded` 必须跟着 `set()` 变。正文一条不许删，分节即可（数据与规模 / 图集 / 攻略帖 / 地图与清单 / 任意地址定位 / 底图与界线合规）。
- 定位层（v9）：路由 `#/at/纬度/经度/缩放/名字`（`navigateAt` + `applyRoute` 的 `state.at`，`lastLoc` 记住来源），地图上落一枚 `.mk-pin` 金色针（和景点光点分开），侧栏给最近 5 座收录城市。**解析顺序固定**：① 经纬度 / 收录城市全名（`Geo.coords` / `Geo.exactLocal`，0 请求）→ ② 本站城市近似（`Geo.localHint`，离线）→ ③ 联网（`Geo.online`：Photon 主、Nominatim 备）。联网只允许在 Enter / 点「📍 定位」时发**一次**，`lastAsk` 保证 ≥1.1s，结果写 `localStorage.cta.geo.v1`（60 天），请求带 `credentials:'omit'` + `referrerPolicy:'no-referrer'`；**不许做输入即搜的联想**。Photon 参数里**不要加 `lang=zh`**（会 400）；`ZOOM_OF` 要同时覆盖 Nominatim 的 `addresstype` 与 Photon 的 `type` / `osm_value`（`train_station` / `apartments` …）。解析失败就退回城市近似并在侧栏写明原因，**不许假装定位成功**。探针钩子 `window.__CTA_GEO`（`coords / local / exact / cached / clear / last / locate`），`verify-v9` 依赖它。
- 站内 AI（v7；v12.1 升级本地模型）：`AI_PROV` 服务商预设 + `AI_LOCAL_MODELS` 本地档位（**Qwen3.5 0.8B / 2B / 4B，默认 2B；Qwen3 1.7B 兼容档**）；请求走 `aiStreamChat()`（`fetch` + SSE 解析 `aiReadSSE()` + `AbortController` 可停），本地模型走 `aiLoadLocal()` → `aiImportWebLLM()`（**WebLLM 0.2.86**，只在点击时 `import()`）→ `aiLocalAppConfig()` → `CreateMLCEngine()`。引擎按 `AI_ENGINE_URLS`（`esm.sh` → `esm.run` → `jsDelivr`）回退；权重 / wasm 用 `AI_LOCAL_SOURCES` 覆盖，默认 `hf-mirror.com` + `gh-proxy.com`（备用 `ghproxy.net`），`auto` 档在 `Failed to fetch` 时再换 HuggingFace / GitHub 官方源（面板 `#aiLocalSrc` 可手动选）。`aiCheckWebGPU()` 先检查 adapter，`aiLocalError()` 区分引擎 / 权重 / wasm / WebGPU / 显存并给处理建议。**首屏零外链**：页面里不许出现 `<script src>`，`check.mjs [14]` 与 `verify-v7.mjs` 都会拦；探针 `window.__CTA_AI` 额外暴露 `localInfo() / localSources() / engineUrls() / webllmVersion()`。
- 城市榜单（v6 / v7 / v10 / v12）：`CN_LISTS` + `CN_LISTS_V7` + `CN_LISTS_V10` + `CN_LISTS_V11` + `CN_LISTS_V12` 必须**一对一覆盖**数据里全部中国城市（**115 城**，`check.mjs [13]` / `[14]` 与 `verify-v7` 会逐城比对，多一个少一个都报错）；条目结构固定为 `routes[名称, 天数, [节点…], 说明] / eat[名称, 说明] / must[名称, 说明]`，节点与景点同名时界面自动变成可点链接（`poiByName()`）。
- 票价（v10）：新增点位统一带 `fee` 字段，侧栏行显示金色徽标、抽屉显示「票价 ¥xx」与「门票」参考价、行程文本在点位后括注；**票价是公开参考价**，淡旺季 / 联票 / 预约政策都会变，界面上必须同时写「以景区当日公示为准」。早期（v4–v9）点位没有逐条标价，抽屉里如实写「未标注（以景区官方渠道为准）」，**不许编造价格**。补点选的是抖音 / 小红书 / B 站上常年有人拍、有人去的公开景点或街区，不写「避雷」向内容，也不抄平台正文。

## v10 本轮（2026-10）

用户的三条反馈都落在这一轮：

1. **铁路网不完整** → `CN_RAIL_LINES` 40 → **71 条**（新增 31 条普速主干 / 高铁通道，含陇海、包西、京通、郑渝、汉十、赣深、贵南、福厦、张吉怀等），并把新城市与延安接进线里；`CN_CITY_NET` 95 → 100 城。
2. **行程助手锁死在南京、改不了** → ① 目的地顶部固定「已选目的地」置顶条，已选城市 / 搜索筛选都能一键取消；② 每个地点旁 `⇄` 换一个 / `✕` 不去，`#plUndo` 恢复；③ 排除项随 `cta.plan.v1` 持久化。验收场景 `v10PlanEdit` 会依次点北京、取消南京、换一个、不去。
3. **每城景点补到 15 个 + 收费景区报价** → 95 座老城各 +5（475 个），新增天水 / 酒泉 / 嘉峪关 / 赤峰 / 克拉玛依 5 城（各 15 个，共 75 个）；中国 100 城 / 1500 景点，全站 159 城 / 1795 景点。550 个新点位**全部带参考票价**（`免费` 或 `¥xx`）。

数据入口：`tools/data/cn-v10-00-new-cities.mjs`（5 座新城 + 榜单）、`cn-v10-01-north.mjs` … `cn-v10-06-northwest.mjs`（95 城补点）→ `node tools/gen-cn-v10.mjs` → `V10-DATA` 标记区。**旧点位不改、不重新编号**；新城市由 `CN_PROV_V10` 补省份归属，运行时 `Object.assign(CN_PROV, …)`。

## v11 本轮（2026-10）

1. **加 10 座城 / 150 个点位**：平凉、庆阳、汉中、安康、上饶、许昌、赣州、郴州、宜宾、泸州，每城 15 个点位；中国 **110 城 / 1650 景点**，全站 **169 城 / 1945 景点**。
2. **铁路网接新城 + 线形改曲线**：`CN_RAIL_LINES` 71 → **73 条**（补宝中铁路接平凉、绵泸高铁接泸州；上饶 / 许昌 / 汉中 / 安康 / 庆阳 / 赣州 / 郴州 / 宜宾改成真实城市站点），`CN_CITY_NET` 100 → **110 城**；渲染用 `smoothPath()` 画 Catmull-Rom 平滑曲线，站点仍按原始坐标画。
3. **行程按距离排**：城际最近邻 + 2-opt；城内先最近邻 + 2-opt，再从热度 / 偏好排序里按最远点选中心、最近点配对后补满，按天分簇；夜景全局挂到最近的收尾点；每天报点位跨度，超过建议值给出提示；同一景点仍然 `dups=0`。
4. **路线地图改真实底图**：`#plMap` 用高德瓦片（GCJ-02）+ Web Mercator，显示前做 WGS-84 → GCJ-02 偏移；保留「全国概览 / 路线范围」切换；瓦片失败时仍显示路线。

## v12 本轮（2026-10）

1. **距离优先再进一步**：v11 的「最近邻 + 2-opt」保留，新增 `plTravelMin()` / `plRouteMinutes()` / `plBucketPolish()`——同一天的点位先分簇，再在两天之间交换 / 移动一个点，只要「两天路上时间之和」变小就接受；每天同时报点位跨度 km 与预计交通分钟，超过节奏建议（60 / 90 / 120 分钟）或单段 >35 km 提醒。
2. **夜市 / 步行街 / 巷子里的宝藏小吃**：新增 `bites` 类别 + 40 个手写点位（`tools/data/cn-v12-02-bites.mjs`），每行 12 列（…|类型|招牌|平台）；没有手写食巷的城市从已有 `food` / `street` 里自动补一个 `p.bite`，保证 115 城都有入口。行程里 `biteQ` 一天最多一个、整趟不重复；结果区新增「🍢 夜市 & 宝藏食巷」清单，平台只给抖音 / 小红书 / B 站**搜索外链**，不抓取正文或视频。
3. **12306 实时余票（可选）**：`tools/12306-proxy.mjs` 只监听 `127.0.0.1:8788`，先取 init Cookie，再转发 `leftTicket/query`；页面 `plLegInfo()` + `plRailQuery()` 按出发日期逐段查询，显示车次 / 时刻 / 历时 / 二等·一等·商务·无座。没启动代理时如实提示「未连接」，仍用站内干线估算；12306 接口无 ACAO，**禁止**在页面里直连。
4. **数据继续扩容**：新增淄博 / 佛山 / 柳州 / 漳州 / 台州 5 城（75 个点位）+ 40 个食巷点位；中国 **115 城 / 1765 景点**，全站 **22 国 / 174 城 / 2060 景点**；`CN_CITY_NET` 补到 115 城（淄博—胶济、佛山西—贵广、柳州—湘桂 / 焦柳、漳州—福厦、台州—杭深）。

## v13 本轮（2026-10）

1. **景点级实用信息**：`CN_PRACTICAL` 首批 51 条（高频 5A / 博物馆 / 山岳 / 海岛），字段 `fee / open / closed / closedDays / reserve / reserveNote / duration / official`；`applyPractical()` 只补字段、不改老数据。没有收录的点位必须显示「未收录（免费 / 收费均有可能）」，不能默认免费。`closedDays` 用 `0=周日 … 6=周六`。
2. **日期与闭馆检查**：`plDateParts()` 从 `plStart` 出发日期换算逐日 `date / label / wd`；命中 `closedDays` 只在顶部 `.pl-closed` 横幅和当日 `.pl-warn` 提示，**绝不自动改行程**。没有填日期时按今天起算，并在说明里写「不填就按今天」。
3. **换城日减负**：`plBuild()` 用 `transferKm` 给换城首日设上限 —— `≥300 km` 当天不排白天点、`120–299 km` 只留 1 个；夜间点位也不能在换城日兜底，排不下就写入 `transferDropped` 并在结果里说明。不要用「压缩档期」静默删点。
4. **预算与住宿**：`plBudget()` 按门票 / 餐饮 / 市内 / 城际 / 住宿拆项；门票只累加已收录价格，`ticketKnown / ticketUnknown` 分开计数；住宿按 `stayOf().band`（双人/间夜）÷2 均摊，合计明确写「2 人同行、1 间房、不含往返大交通」。`plStayHtml()` 的晚数来自 `days.slice(0,-1)` 的实际过夜城市。
5. **可访问性 / 结果定位**：`trapTab()` 让抽屉与行程面板 Tab 不跑到底层；关闭后焦点回到触发元素；点「生成行程」后 `#plOut.scrollIntoView`。
6. **验证**：`node tools/check.mjs` 的 [20]、`node tools/verify-v13.mjs`、`node tools/probe-v2-all.mjs v13Plan v13Close` 必须全绿；`v13Close` 专门断言「周一闭馆只提示、故宫仍在第 1 天」。
7. **数据边界**：v13 不扩城市 / 景点（用户点名的体验轮，v8 / v9 同类例外）；下一轮恢复「每城 ≥15 个点位」扩容。

## 下一步

1. 下一轮继续按 **每城 ≥15 个点位**补城：优先补尚未覆盖的地级市，每城 15 个，走 `cn-v12-*.mjs` 同一格式（新文件用 `cn-v13-*.mjs` + 新生成器标记区，别覆盖 v12 手稿）。
2. 早期点位补票价：先挑 5A / 4A 收费景区逐条补 `fee`（仍按「参考价 + 以景区公示为准」口径），不要给免费街区硬编价格。
3. 铁路网下一轮补支线时，先跑 `node tools/gen-cn-v8.mjs --dry` 看站点是否都在版图内；新城市必须先跑最新城市生成器，再跑 v8 生成器。

