# 工业设备状态监控与数字孪生系统 (Digital Twin Monitoring System)

## 项目简介
基于 Vue3 + Three.js + MQTT + InfluxDB + Grafana 构建的工业设备实时监控与数字孪生系统。模拟产线 3 台核心设备（电机、泵、风机）实时上报温度、振动、电流数据，前端通过 WebSocket 接收数据流，驱动 3D 模型根据阈值实时变色（正常绿色、预警黄色、告警红色），同时通过 Grafana 展示 2D 数据看板。

## 技术栈
- **前端**: Vue3 + Vite + Three.js + MQTT.js
- **后端/数据**: Python (paho-mqtt) + Mosquitto (MQTT Broker) + InfluxDB (时序数据库)
- **可视化**: Grafana + Three.js Web3D
- **操作系统**: Ubuntu 22.04 LTS (虚拟机) + Windows (宿主开发环境)

## 系统架构
`Python 设备模拟器` -> `MQTT Broker (Mosquitto)` -> `(订阅端写入 InfluxDB -> Grafana 2D看板)` & `(WebSocket -> Vue3 3D数字孪生变色)`

## 核心功能与排错记录
1. **数据驱动 3D 渲染**：建立设备 ID 与 3D 网格 (Mesh) 的映射表，根据实时温度数据（>80℃红色告警，60-80℃黄色预警）动态修改材质颜色。
2. **WebSocket 跨主机通信**：打通 Windows 宿主机与 Linux 虚拟机之间的网络隔离，配置 Mosquitto WebSocket 监听端口 (9001) 并放行防火墙，实现浏览器端实时数据订阅。
3. **模型解码与 CORS 跨域**：解决本地 file:// 协议加载模型被 CORS 拦截问题，并引入 MeshoptDecoder 解决压缩模型解析报错。
4. **前端工程化重构**：从原生 HTML 重构为 Vue3 工程，将 3D 渲染与 MQTT 逻辑封装为独立组件（TwinScene.vue），利用 onMounted / onUnmounted 生命周期精准管理资源释放，防止内存泄漏。

## 快速开始
1. 启动 Mosquitto、InfluxDB、Grafana 服务
2. 运行 Python 模拟器: `python3 device_simulator.py`
3. 安装前端依赖: `npm install`
4. 启动 Vue 项目: `npm run dev`
5. 访问 `http://localhost:5173/` 查看 3D 数字孪生看板

## 项目截图

## 1.终端数据流
<img width="773" height="942" alt="ScreenShot_terminal" src="https://github.com/user-attachments/assets/79b46d94-c590-4842-ab40-2ccd478abc9e" />


## 2.Grafana 监控看板 - 总览
<img width="1659" height="1308" alt="ScreenShot_Dashboards_normal" src="https://github.com/user-attachments/assets/4b636cec-7b57-4f92-9906-e23f300b0628" />


## 3. Grafana 监控看板 - 告警与指标分析
<img width="891" height="426" alt="ScreenShot_alert2" src="https://github.com/user-attachments/assets/4c22bf26-4f49-4c44-b1c7-bc35482670a6" />
<img width="929" height="453" alt="ScreenShot_alert_recover" src="https://github.com/user-attachments/assets/7e8cc468-114b-4c14-a8be-a6ca82c48c3c" />
<img width="915" height="435" alt="ScreenShot_alert_firing" src="https://github.com/user-attachments/assets/477f86d3-7ba0-49f0-8100-e29009e52c43" />


## 4. 3D 数字孪生设备状态视觉映射对比
<img width="1164" height="527" alt="ScreenShot_normal_model" src="https://github.com/user-attachments/assets/88b8fec2-1744-46fd-b2dd-27b676e0a840" />
<img width="897" height="474" alt="ScreenShot_alert_models" src="https://github.com/user-attachments/assets/4fb50a9b-b72e-45eb-a6a2-0cc63398fbc4" />


   
