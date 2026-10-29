<template>
  <div class="map-container">
    <!-- 装地图的容器 -->
    <div ref="chartDom" style="width: 100%; height: 500px;"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as echarts from 'echarts'
// 引入中国地图数据 (最新版 echarts 的引入方式)
import chinaMapData from 'echarts/map/json/china.json'

const chartDom = ref(null)
let myChart = null

// 定义向上抛出的事件
const emit = defineEmits(['map-click'])

onMounted(() => {
  // 1. 注册地图
  echarts.registerMap('china', chinaMapData)
  // 2. 初始化图表
  myChart = echarts.init(chartDom.value)

  const option = {
    title: { text: '🇨🇳 中国旅游足迹图', left: 'center', textStyle: { color: '#333' } },
    tooltip: { trigger: 'item', formatter: '{b}' }, // 鼠标悬浮显示省份名
    geo: {
      map: 'china',
      roam: true, // 允许缩放和拖拽
      zoom: 1.2,  // 初始放大一点
      label: { show: true, color: '#fff' },
      itemStyle: {
        areaColor: '#3b82f6', // 默认省份颜色
        borderColor: '#fff',
        borderWidth: 1
      },
      emphasis: { // 鼠标悬浮高亮效果
        itemStyle: { areaColor: '#f59e0b' },
        label: { show: true, color: '#fff' }
      }
    }
  }

  myChart.setOption(option)

  // 3. 绑定点击事件
  myChart.on('click', (params) => {
    if (params.componentType === 'geo') {
      // 触发事件，把省份名字传给父组件
      emit('map-click', params.name)
    }
  })
})

// 组件销毁时释放内存
onBeforeUnmount(() => {
  if (myChart) myChart.dispose()
})
</script>

<style scoped>
.map-container {
  background: #fff;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  margin-top: 20px;
}
</style>