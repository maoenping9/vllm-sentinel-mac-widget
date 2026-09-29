# vLLM Sentinel — macOS 桌面 3D 机箱组件

将 vLLM Sentinel 控制台的关键指标以**等轴 3D 机箱**实时渲染在 Mac 桌面上（壁纸之上、窗口之下），2 秒刷新。适配 Intel / Apple Silicon Mac。

**组件版本 v97.2**（随后端 vLLM Sentinel **v2.2** 发布）。

## ⚙️ Mac 需要安装的软件（先装这些）

| 软件 | 用途 | 安装命令 |
|------|------|----------|
| **Übersicht** | 桌面 widget 框架（必装，本组件运行环境） | `brew install --cask ubersicht` 或官网 http://tracesof.net/uebersicht/ |
| **Homebrew** | 包管理（装 Übersicht 的前提） | 官网 https://brew.sh：`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` |
| **Git**（可选，用于拉仓库） | 下载源码 | `brew install git`（macOS 自带 `/usr/bin/git` 也行） |
| **curl**（系统自带） | 组件内部取数，无需安装 | macOS 自带 |

> 简版：装好 **Homebrew** → 然后 **`brew install --cask ubersicht`** → 就只剩放 widget 文件夹了。

## 依赖：vLLM Sentinel 后端控制台

本组件本身不采集数据，它只读后端控制台提供的 `GET /api/state` 与 `GET /api/energy`。

- 后端 Web 控制台源码（FastAPI + React/Vite，一键 `docker compose up -d` 拉起）：
  **https://github.com/promisezackr/vllm-sentinel**
- 本组件对应的最新后端版本（v2.2，含下面的「流量累计/月度年度累计」新字段）在同仓库 fork：
  **https://github.com/maoenping9/vllm-sentinel**

组件通过 Mac 本地 `curl` 访问该控制台（无 CORS 问题），在 `index.jsx` 顶部配置服务器地址即可。

> 兼容性：旧版后端（无 `net_cycle_bytes` 等字段）也能用 —— 组件会回退到「瞬时速率 × 周期已过时长」的旧算法，
> 只是累计值会随速率跳动、流量口径为全部网卡。

## 3D 机箱展示内容（v2.2）

- **卡数自适应**：卡条/风扇/卡号跟随本机实际 GPU 数量（15/16 卡也不会排出机箱底），加卡自动跟上
- **GPU 涡轮风扇**（9733）：大圆形轮毂 + 放射密排叶片，转速实时（停转/失联红叉）
- **Noctua 猫扇阵列**：顶部 3×12NF、前面 6×G1 + 1×前NF、后面 5×G2 排风，转速实时
- **显卡条**：显存段（模型配色/空闲灰/过热橙红）+ 功率段 + 温度段，CPU/模型 ↔ GPU 实时对照
- **气流动画**：前端进风 → GPU 卡仓 → 后排 G2 排出的跳动虚线流向
- **等轴旋转**：机箱在 -26°↔-50° 之间缓慢摆动，各面/风扇/编号遮挡正确
- **内存**：CPU 区内存使用率条 + 内存条温度 0/1 + 内存供电温度 0/1（来自 BMC 传感器，按传感器自带阈值上色）
- **网络流量（20 日起）**：读后端**持久累加值**（单调递增、不再跳动）；三个额度条分别对应本月电费 / 本年电费 / 当代流量周期
- **模型命名对齐**：DeepSeek V4.1（裸名）与 Qwen3.8 家族（INT8 / Flash-Next / W4A16）在组件与 Web 控制台同名同色
- **附加监控**：模型在线/吞吐/KV 缓存、CPU 逐核、磁盘 IO、整机功耗与电源额定

### 转速是怎么来的

组件不直接量转速，而是按控制器档位换算（满速 = 3450 RPM）：

| 来源 | 换算 | 说明 |
| --- | --- | --- |
| Corsair Commander（`pwmN`） | `PWM / 255 × 3450` | 由系统 hwmon 读取，后端换算后经 `/api/state` 下发 |
| BOSS Storm3 串口风机盒 | `档位 / 127 × 3450` | 后端读串口 `get_LEVELS`；一台机器若挂多只同型盒，按官方驱动节点当前指向自动选口 |
| 无风扇源的卡 | `--` / 0 | 例如显卡自身零风扇模式（低负载停转），属真实读数 |

## 组件常量（`Übersicht/widgets/vllm-sentinel.widget/index.jsx` 顶部）

| 常量 | 默认 | 说明 |
|------|------|------|
| `SERVER` | `http://your-server-ip:8889` | 后端控制台地址（**必改**） |
| `REFRESH_MS` | `2000` | 刷新间隔（毫秒） |
| `HOT_TEMP` | `75` | 达到该温度变红（GPU 负载条 / CPU 温度） |
| `NET_TOTAL_TB` | `1.5` | 流量周期总配额（TB） |
| `NET_WARN_TB` | `1.0` | 周期累计达到后数值与进度条变红 |
| `NET_CYCLE_START_DAY` | `20` | 流量周期起始日，**需与后端 `NET_CYCLE_START_DAY` 一致** |
| `OTHER_W` | `200` | 整机功耗估算中「GPU 之外」的固定补偿（瓦） |
| `NET_USED_GB_OVERRIDE` | `250` | 仅旧后端兜底时的手工基准（GB）；接 v2.2 后端后改由后端基线控制 |

## 安装（Mac）

1. 装好上面表格里的 **Übersicht**（及 Homebrew）
2. 克隆本仓库（或用 Git 下载 zip），把 `Übersicht/widgets/vllm-sentinel.widget` 整个文件夹拷贝到：
   `~/Library/Application Support/Übersicht/widgets/`
3. 打开 `vllm-sentinel.widget/index.jsx` 顶部，把 `SERVER` 常量改成你的
   vLLM Sentinel 控制台地址（默认占位 `http://your-server-ip:8889`）
4. Übersicht 会自动加载；如没刷新，在菜单栏 Übersicht 图标里点 Refresh（或 `Cmd+R`）

### 可选：浮动监控窗（desktop/floating）

`vllm-hud.js` + `vllm-monitor.html` 是桌面轻量浮动窗（非 3D），供口味偏好简洁时使用：
把 `vllm-monitor.html` 放在 Mac 本地目录，编辑 `vllm-hud.js` 里的路径与 `SERVER` 后，
用 `osascript` 或快捷键调起即可。核心 3D 组件不受此影响。

## 数据安全说明

发布版已移除所有内网地址、主机名、端口、CUDA 分配与模型启动脚本细节，
`SERVER` 为占位符，你需填入自己的控制台地址。

## License

MIT
