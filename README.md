# 工业物联网监控可视化大屏
> IoT Dashboard | Vue3 + Vite + ECharts + MQTT + Three.js

## ✨ 项目介绍
本项目是工业物联网可视化监控大屏Demo，面向工业设备监控场景。
- 左侧：设备在线/离线、告警数量统计卡片，实时刷新
- 中间：ECharts实现温度趋势折线图、设备类型分布饼图，自适应窗口
- 右侧：Three.js封装基础3D设备预览组件，支持鼠标拖拽旋转、滚轮缩放
- 实时数据流：集成mqtt.js，基于MQTT over WebSocket订阅设备上报Topic，包含消息解析、异常捕获、断线重连逻辑
- 本地演示：使用定时器模拟传感器实时报文，**无需部署MQTT服务即可直接运行**；切换真实MQTT地址即可接入真实设备数据
- 资源管理：组件销毁时，统一释放ECharts实例、MQTT连接、WebGL渲染资源，避免内存泄漏

> 项目定位：IoT可视化大屏实训Demo，3D模块作为辅助预览，核心聚焦实时数据可视化。

## 🛠 技术栈
- 构建框架：Vue3 + Vite
- 可视化图表：ECharts
- 物联网通信：mqtt.js（MQTT over WebSocket）
- 3D WebGL：Three.js
- 代码规范：Composition API `<script setup>`

## 🚀 在线演示
🔗 Demo地址：https://xxx.vercel.app （等你部署完成替换链接）

## 💻 本地运行
### 环境要求
Node.js >= 18（推荐 Node 22 LTS）

```bash
# 1. 克隆代码
git clone https://github.com/meiqianyao/iot-dashboard.git
cd iot-dashboard

# 2. 安装依赖（不会上传node_modules）
npm install

# 3. 本地开发启动
npm run dev

# 4. 打包构建（部署用）
npm run build

## ✨ 目录结构
iot-dashboard/
├── src/
│   ├── main.js               # 入口文件
│   ├── App.vue
│   ├── views/
│   │   └── Dashboard.vue     # 大屏主页面（MQTT逻辑写在这里）
│   └── components/
│       └── ThreeModel.vue    # Three.js 3D预览组件
├── index.html
├── package.json
├── vite.config.js
├── .gitignore
└── README.md

## 📌 功能说明

1. 实时时钟展示
2. 设备指标卡片动态更新
3. 温度趋势折线滚动刷新
4. 3D 模型交互，点击模型触发数据更新
5. MQTT 客户端代码预留，支持接入真实物联网 Broker
6. 页面自适应窗口大小，图表自动 resize

## ⚠️ 说明

MQTT 连接代码默认注释，本地运行使用 mock 定时器模拟设备推送。
如需接入真实设备，打开注释，修改 Broker 地址与 Topic 即可。

## 📷 项目截图
![IoT可视化大屏预览图](./assets/dashboard.png)
