# vLLM Sentinel — macOS 桌面 3D 机箱组件

将 vLLM Sentinel 控制台的关键指标以**等轴 3D 机箱**实时渲染在 Mac 桌面上（壁纸之上、窗口之下），1 秒刷新。适配 Intel / Apple Silicon Mac。

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

组件通过 Mac 本地 `curl` 访问该控制台（无 CORS 问题），在 `index.jsx` 顶部配置服务器地址即可。

## 3D 机箱展示内容（v2.0）

- **14 卡全量**：13× NVIDIA CMP 170HX + 1× 3090 Ti 逐卡显示
- **GPU 涡轮风扇**（13×9733）：大圆形轮毂 + 放射密排叶片，转速实时（停转/失联红叉）
- **Noctua 猫扇阵列**：顶部 3×12NF、前面 6×G1 + 1×前NF、后面 5×G2 排风，转速实时
- **显卡条**：负载段（显存占用，模型配色/空闲灰/过热橙红）+ 功率段（蓝），CPU/模型↔GPU 实时对照
- **气流动画**：前端进风 → GPU 卡仓 → 后排 G2 排出的跳动虚线流向
- **等轴旋转**：机箱在 -26°↔-50° 之间缓慢摆动，各面/风扇/编号遮挡正确
- **附加监控**：模型在线/吞吐/KV 缓存、CPU 逐核、内存/磁盘 IO/网络、整机功耗与电源额定

## 安装（Mac）

1. 装好上面表格里的 **Übersicht**（及 Homebrew）
2. 克隆本仓库（或用 Git 下载 zip），把 `Übersicht/widgets/vllm-sentinel.widget` 整个文件夹拷贝到：
   `~/Library/Application Support/Übersicht/widgets/`
3. 打开 `vllm-sentinel.widget/index.jsx` 顶部，把 `SERVER` 常量改成你的
   vLLM Sentinel 控制台地址（默认 `http://你的服务器IP:8889`）
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
