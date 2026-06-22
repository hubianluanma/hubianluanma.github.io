+++
date = '2026-06-22T13:44:00+08:00'
draft = false
author = 'Spiral'
title = 'M4 Mac mini 的功耗监测面板：从 powermetrics 到外网可视化仪表盘'
description = "用一个周末时间把 Mac 的耗电量从命令行数字变成可通过 https://hubianluanma.com/power/ 访问的实时 Web 仪表盘，含北京阶梯电价估算。代码已开源：https://github.com/hubianluanma/mac-power-monitor"
tags = ["编程", "技术", "Mac", "Flask", "SQLite"]
categories = ["编程技术"]
[cover]
image = "https://minio-api.hubianluanma.com/typora/2026/06/22/9b430c99-6c8d-4001-8cf1-fc4b14a04159.png"
alt = "Mac 功耗监测仪表盘"
caption = ""
relative = false
hidden = false
+++

我的 M4 Mac mini 24 小时开着，主要跑 Docker、ComfyUI 实验和几个长任务。我想知道它每天到底花我多少钱电费——这种问题，最朴素的做法是查电表，但电表是整户楼的，单独拎出一台机器的耗电得自己测。

于是花了半天时间，从一行 `powermetrics` 命令开始，一步步把它搭成了一个可以通过外网访问的实时 Web 仪表盘。这篇文章不是 step-by-step 教程，而是讲清楚**整个过程中的技术决策和踩坑实录**——如果你也想给自己的 Mac 装一个，可以照着流程做；如果你只是想看看怎么从"命令行工具"演进到"可视化系统"，也许能从我的判断里捞到一些东西。

最终成品在：`https://hubianluanma.com/power/`，完整代码已开源在 [GitHub](https://github.com/hubianluanma/mac-power-monitor)。

## 一、方案选型

Mac 上能拿到实时功耗数据的工具不多，挨个比一遍：

| 工具 | 类型 | 能拿到的数据 | 我的取舍 |
|---|---|---|---|
| `powermetrics` | 系统 CLI（root） | CPU/GPU/ANE 毫瓦级功耗 | ✅ 数据最准，但要 sudo |
| `pmset` | 系统 CLI | 电池放电率、电源事件 | ❌ 桌面机没电池用不上 |
| Stats (开源 GUI) | 菜单栏 app | 实时瓦数 + 温度 | ✅ 装一个看瞬时值 |
| Watts / iGlance | 菜单栏 app | 瓦数 | 同 Stats，没特别优势 |
| Activity Monitor → 能耗 | 系统 GUI | app 级影响 | ❌ 不直观，看不到总瓦数 |

结论：**底层用 `powermetrics`，UI 用 Stats**，再加一层自己的采样 + Web 展示层做历史和成本估算。

## 二、架构

演化到最后的完整结构是这样的：

```
┌─────────────────────┐
│ powermetrics (sudo) │  ← 每分钟采样 5s
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ SQLite 单文件 DB    │  ← /power/data/power.db
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Flask (127.0.0.1)   │  ← 单文件服务，端口 7654
│  /api/summary       │
│  /api/samples       │
│  /api/hourly        │
│  /api/cities        │
└──────────┬──────────┘
           ↓ proxy_pass /power/
┌─────────────────────┐
│ nginx (Docker) :80  │  ← 反代到 Flask
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Cloudflare Tunnel   │  ← HTTPS 终止（已有）
└──────────┬──────────┘
           ↓
     浏览器手机访问
```

**为什么这样分层**：采集层（powermetrics）只关心拿到数据，存储层（SQLite）只关心怎么存，Web 层（Flask）只关心怎么展示。每层都能独立替换或测试。

## 三、采集器：powermetrics + NOPASSWD sudo

`powermetrics` 需要 root 权限，但每分钟采一次，每次还要人输密码就太蠢了。给当前用户配 NOPASSWD sudoers 是常规做法：

```
# /etc/sudoers.d/hermes-powermetrics
spiral ALL=(root) NOPASSWD: /usr/bin/powermetrics
Defaults!/usr/bin/powermetrics !logfile, !syslog
```

关键的两个细节：
1. **限定到具体路径**，不给完整 root NOPASSWD（最小权限原则）
2. **`!logfile, !syslog`** 让这个命令不写系统日志，避免 syslog 被噪声淹没

采集器是个简单的 Python 循环，对齐整分钟采样 5 秒，解析三个 Power 字段：

```python
# CPU Power: 123 mW
# GPU Power: 45 mW
# ANE Power: 0 mW
# Combined Power (CPU + GPU + ANE): 168 mW
```

正则取出最后一次值（最稳定），写 SQLite。表结构只有 7 个字段，单文件 DB，零运维。

**一个小坑**：powermetrics 只给 SoC package 的功耗（CPU+GPU+ANE），不算屏幕/SSD/USB/Wi-Fi/DRAM 这些系统外设。M4 Mac mini 闲置时整机功耗 ~7-9W，但 SoC 端只有 0.1W 不到。差额主要是"系统开销"，我硬编码了一个 `SYSTEM_BIAS_W = 7.0` 加进去当估算——比裸 SoC 数据更接近电表读数。

## 四、Web 仪表盘：Flask + Chart.js

前端只有一页，没有用 React/Vue 这种重武器——单文件 HTML + Chart.js CDN 引一个就够。理由：
- 实时数据靠 `setInterval(refresh, 60000)`，不需要响应式状态管理
- 图表就 3 个，Chart.js 一两百行代码搞定
- 不需要 build pipeline，编辑 `index.html` 改完直接生效

页面有 6 个 KPI 卡片 + 3 个图表 + 1 张阶梯电价表。KPI 之间没有复杂的依赖关系，全部从 `/api/summary` 一个接口拿，省得多发请求。

## 五、电价模块：北京阶梯电价

最有意思的需求是"按北京民电价格估算成本"——北京居民用电是**阶梯电价**，按**年累计度数**分档：

| 档位 | 年累计范围 | 单价 |
|---|---|---|
| 一档 | 0 – 2520 kWh | 0.5469 元/kWh |
| 二档 | 2521 – 4800 kWh | 0.5969 元/kWh |
| 三档 | 4800+ kWh | 0.8469 元/kWh |

一个 Mac 一年撑死跑 100-200 度电，永远落在一档，但**算法要写对**——按档位逐档累加，不能直接拿总度数乘一档单价，否则跨档时算错。

后端算 `year_kwh_est`（用本年日均功率 × 已过年天数估算），前端根据这个数逐档累加算出总费用，把"当前落在哪档"高亮出来。一台 Mac 的实际数字：年估算 ~50 kWh / 年费用 ~27 元——比想象中便宜。

仪表盘顶部还做了**城市切换**，支持北京、上海、广州、深圳、统一价 0.6 元——出差或迁居时改一下就生效。

## 六、外网访问：nginx 反代 + Cloudflare Tunnel

**前提**：我已经有 Cloudflare Tunnel 把 `hubianluanma.com` 转到本地 nginx（Docker 容器，80/443）。所以新功能只需要在 nginx 配一段反代就行，HTTPS 不用管。

nginx 这段配置很标准：

```nginx
location /power/ {
    proxy_pass http://host.docker.internal:7654/power/;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_buffering off;
    proxy_cache off;
    add_header Cache-Control "no-store, no-cache, must-revalidate" always;
}
```

三个关键点：
1. **`proxy_pass` 末尾带 `/power/`** —— 让 nginx 把 `/power/api/x` 转成 Flask 的 `/power/api/x`，Flask 路由直接挂 `/power/` 前缀
2. **`host.docker.internal:7654`** —— Docker Desktop Mac 让容器访问宿主机的方式
3. **`Cache-Control: no-store`** —— 防止 Cloudflare 缓存实时 JSON

第二个 `Cache-Control` header 也要在 Flask 端再加一层（`@app.after_request`），因为 nginx 的 `add_header` 会被上游响应覆盖，双保险。

## 七、踩坑实录

这一节才是真正的内容——上面的"方案"是事后看起来很顺，但实现过程有几个坑值得记下来。

### 坑 1：Flask 路由前缀的两种思路

Flask 接到 `/power/api/x` 请求有两种处理方式：
- **A. 用 `SCRIPT_NAME` 环境变量**（WSGI 标准做法，但单独 Flask app 不会自动改 URL map）
- **B. 直接把路由挂到 `/power/api/...`**

我先按 A 写了，结果所有路由都 404。Flask 文档里要装 `DispatcherMiddleware` 或者用 `werkzeug.middleware.proxy_fix.ProxyFix` 才能让 `SCRIPT_NAME` 生效——单文件 Flask 不行。**改用 B，所有路由加前缀，反而更简单**。

### 坑 2：hermes 沙盒启服务

整个开发流程在 Hermes Agent 里跑（沙盒执行环境），它对后台启进程的处理跟普通 shell 不一样。`subprocess.run()` + 后台 `&` 启 Flask 时，进程虽然起来了，但 stdio 被劫持，curl 拿不到响应——socket 状态是 `LISTEN`，但 accept 不工作。

**解决方法**：用 Hermes 的 `terminal(background=true)` 工具，它会自动接管进程生命周期，让 `PATH` 里 `/opt/homebrew/bin` 排第一（避免 hermes venv 的 Python 3.11 没装 flask）。

调试这种"沙盒内服务"问题的快速诊断：看 `lsof -i :7654` 显示的状态——`LISTEN` 是正常，`CLOSED` 是 socket 已关，`TIME_WAIT` 是连接刚断。如果一直是 `CLOSED` 但 PID 活着，说明 `bind()` 后被立刻 close，多半是 launchd 或沙盒的策略拦截。

### 坑 3：JS fetch().json() 异步链

调通后第一次刷新页面，右上角报：

```
连接失败: Cannot read properties of undefined (reading 'tiers')
```

错误是前端 tier 表渲染里写的：

```js
// 错的
const cityConf = (await fetch(API('/api/cities')))[currentCity];
```

`fetch()` 返回的是 `Response` 对象，**不是 JSON**。所以 `[currentCity]` 取的是 `undefined.tiers`。正确写法：

```js
// 对的
const allCities = await (await fetch(API('/api/cities'))).json();
const tiers = allCities[currentCity].tiers;
```

这种 bug 在异步代码里很常见，因为 `Response` 也有 `[...]` 这种索引行为（虽然没意义），TypeScript 能抓但纯 JS 写时容易忽略。

### 坑 4：Flask 模板缓存

修完这个 bug 后，前端还是报同样错。看 served HTML：

```bash
curl http://127.0.0.1:7654/power/ | grep cityConf
# 还是老代码，不是新的 allCities
```

但磁盘上的文件已经改了。Flask 在 `debug=False` 模式下，Jinja2 `auto_reload` 是开的，但 Py3.14 + Werkzeug 3.1 这套组合下偶尔检测不到 mtime 变化。**最稳的做法：改模板后 kill 进程重启**——单文件 Flask 不值得折腾 `extra_files`。

## 八、当前数据和总结

跑了一上午的实测数字：

- **当前功耗**：7-10 W（闲置）
- **今日累计**：~0.05 kWh / 0.03 元
- **每月估算**：~4 kWh / 2 元
- **每年估算**：~50 kWh / 27 元（落一档电价）

比我预期的便宜——M4 Mac mini 真的省电。但有意思的是数据里有一次明显的 spike：12:44:06 那分钟 CPU 跳到 3W（平时 0.05W），是某次 spotlight 索引或者其他系统任务触发的，平时完全看不到这种细节。

整套系统包含 3 个文件共 ~300 行 Python + ~400 行 HTML + ~30 行 nginx 配置：
- `~/power-monitor/scripts/collector.py` — 数据采集
- `~/power-monitor/scripts/web.py` — Flask 服务
- `~/power-monitor/templates/index.html` — 前端单页

**做完这个项目最大的收获**：把一个看似简单的"我想看耗电"需求，认真当成一个完整的小系统来做——采集、存储、展示、外网访问、成本估算，每个层次都值得用最合适的工具，而不是一刀切用同一个 stack。当你真正搭起来后，下次想看任何机器的耗电都只需要改一行配置。

---

## 开源地址

整套代码已经整理好放上 GitHub：`https://github.com/hubianluanma/mac-power-monitor`

包含的内容（自项目化重构后比本文写得更完整）：

- **双 README**（中英）+ MIT License
- **代码参数化**：`SYSTEM_BIAS_W` / `PORT` / `SCRIPT_PREFIX` 都通过环境变量配，不同机型（M 系列 MacBook / Mac mini / Mac Studio）直接改一行就能用
- **nginx 反代配置示例**（`examples/nginx.conf`），含 Cloudflare 缓存绕过说明
- **社区文件**：Issue 模板、PR 模板、FUNDING

如果你装了发现 bug 或者想加新功能（比如支持 Windows 的 `powercfg`、加 Grafana 集成、做 PWA 离线），欢迎在 repo 提 Issue 或 PR。

---

*如果你也想给自己的 Mac 装一套，可以照着这个流程走；遇到具体的坑欢迎在评论区交流。*