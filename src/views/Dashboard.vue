<template>
  <div class="iot-dashboard">
    <!-- 头部标题 -->
    <div class="header">
      <h1>工业物联网设备监控大屏</h1>
      <div class="time">{{ currentTime }}</div>
    </div>

    <div class="content-wrap">
      <!-- 左侧统计面板 -->
      <div class="left-panel">
        <div class="stat-card">
          <div class="label">在线设备</div>
          <div class="value">{{ deviceOnline }}</div>
        </div>
        <div class="stat-card">
          <div class="label">离线设备</div>
          <div class="value">{{ deviceOffline }}</div>
        </div>
        <div class="stat-card">
          <div class="label">告警数量</div>
          <div class="value warn">{{ alertCount }}</div>
        </div>
      </div>

      <!-- 中间图表区域 -->
      <div class="center-panel">
        <div ref="lineChartRef" class="chart-item"></div>
        <div ref="pieChartRef" class="chart-item pie"></div>
      </div>

      <!-- 右侧3D模型区域（Three.js数字孪生组件） -->
      <div class="right-panel">
        <ThreeModel @deviceClick="handleDeviceClick"/>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'
import mqtt from 'mqtt'
import ThreeModel from '../components/ThreeModel.vue'

// 页面状态数据
const currentTime = ref('')
const deviceOnline = ref(126)
const deviceOffline = ref(14)
const alertCount = ref(3)

let timer, lineChart, pieChart, realDataTimer
let mqttClient = null
const lineChartRef = ref(null)
const pieChartRef = ref(null)

// ===================== MQTT 相关配置与方法 =====================
const mqttOptions = {
  clientId: 'iot_dashboard_' + Math.random().toString(16).slice(2)
}
// 真实MQTT服务地址，前端使用 MQTT over WebSocket
// const brokerUrl = 'ws://127.0.0.1:8083/mqtt'
const topic = 'device/sensor/data'

// MQTT初始化连接
function initMqtt() {
  // mqttClient = mqtt.connect(brokerUrl, mqttOptions)

  // 连接成功回调
  // mqttClient.on('connect', () => {
  //   console.log('MQTT连接成功，订阅topic：', topic)
  //   mqttClient.subscribe(topic)
  // })

  // 收到设备推送消息
  // mqttClient.on('message', (topic, payload) => {
  //   try {
  //     const msg = JSON.parse(payload.toString())
  //     updateDashboardData(msg)
  //   } catch (err) {
  //     console.error('MQTT消息解析失败', err)
  //   }
  // })

  // 连接异常
  // mqttClient.on('error', (err) => {
  //   console.error('MQTT连接异常：', err)
  // })

  // 连接断开，自动重连
  // mqttClient.on('close', () => {
  //   console.log('MQTT连接断开，3s后尝试重连')
  //   setTimeout(() => initMqtt(), 3000)
  // })
}

/**
 * MQTT消息到达后，更新大屏数据
 * @param {Object} msg 设备上报数据 { temp, online, offline, alert }
 */
function updateDashboardData(msg) {
  deviceOnline.value = msg.online
  deviceOffline.value = msg.offline
  alertCount.value = msg.alert

  // 更新温度折线图，滚动数据（去掉最早一条，追加新数据）
  const oldData = lineChart.getOption().series[0].data
  oldData.shift()
  oldData.push(msg.temp)
  lineChart.setOption({
    series: [{ data: oldData }]
  })
}

// ===================== 大屏图表、模拟数据逻辑 =====================
// 更新页面时钟
function updateTime() {
  const now = new Date()
  currentTime.value = now.toLocaleString()
}

// 折线图：设备温度趋势
function initLineChart() {
  lineChart = echarts.init(lineChartRef.value)
  const option = {
    title: { text: '设备温度趋势' },
    tooltip: { trigger: 'axis' },
    xAxis: {
      type: 'category',
      data: ['00:00','04:00','08:00','12:00','16:00','20:00']
    },
    yAxis: { type: 'value', name:'℃', min:20, max:40 },
    series: [{
      name:'温度',
      type:'line',
      data:[24,26,30,34,29,25],
      smooth:true
    }]
  }
  lineChart.setOption(option)
}

// 饼图：设备类型分布
function initPieChart() {
  pieChart = echarts.init(pieChartRef.value)
  const option = {
    title:{text:'设备类型占比'},
    tooltip:{trigger:'item'},
    series:[{
      type:'pie',
      radius:'60%',
      data:[
        {value:60,name:'传感器'},
        {value:40,name:'电机'},
        {value:25,name:'阀门'},
        {value:15,name:'仪表'}
      ]
    }]
  }
  pieChart.setOption(option)
}

// 模拟IoT后端推送实时数据（本地演示用，真实环境替换为MQTT消息）
function refreshRealData() {
  const mockMsg = {
    temp: Math.max(20, Math.min(40, 25 + Math.random() * 10)),
    online: deviceOnline.value + (Math.floor(Math.random() * 3) - 1),
    offline: deviceOffline.value - (Math.floor(Math.random() * 3) - 1),
    alert: alertCount.value + (Math.random()>0.7 ? 1 : 0)
  }
  updateDashboardData(mockMsg)
}

// 3D模型点击事件：联动刷新图表
function handleDeviceClick() {
  refreshRealData()
}

// 窗口自适应
function resizeAll() {
  lineChart?.resize()
  pieChart?.resize()
}

onMounted(()=>{
  updateTime()
  timer = setInterval(updateTime,1000)
  initLineChart()
  initPieChart()
  window.addEventListener('resize', resizeAll)

  // 初始化MQTT（当前内部逻辑注释，不会建立连接）
  initMqtt()
  // 本地演示：定时器模拟实时消息推送
  realDataTimer = setInterval(refreshRealData,3000)
})

onUnmounted(()=>{
  // 清除定时器
  clearInterval(timer)
  clearInterval(realDataTimer)
  window.removeEventListener('resize', resizeAll)

  // 销毁echarts实例
  lineChart?.dispose()
  pieChart?.dispose()

  // 关闭MQTT连接，释放资源
  if(mqttClient) {
    mqttClient.end()
    mqttClient = null
  }
})
</script>

<style scoped>
.iot-dashboard{
  width:100%;
  height:100vh;
  background:#0f172a;
  color:#fff;
  padding:16px;
  box-sizing:border-box;
}
.header{
  display:flex;
  justify-content:space-between;
  align-items:center;
  font-size:22px;
  margin-bottom:16px;
  h1 {
    color: #fff;
  }
}
.content-wrap{
  display:grid;
  grid-template-columns:20% 1fr 35%;
  gap:16px;
  height:calc(100% - 60px);
}
.left-panel{
  display:flex;
  flex-direction:column;
  gap:16px;
}
.stat-card{
  background:rgba(30,41,59,0.7);
  padding:20px;
  border-radius:8px;
  border:1px solid #2563eb;
}
.label{
  font-size:14px;color:#94a3b8;
}
.value{
  font-size:32px;font-weight:bold;margin-top:8px;
}
.warn{
  color:#f87171;
}
.center-panel{
  display:flex;
  flex-direction:column;
  gap:16px;
}
.chart-item{
  flex:1;
  background:rgba(30,41,59,0.7);
  border-radius:8px;
}
.pie{
  flex:0.8;
}
.right-panel{
  height:100%;
}
</style>
