<template>
  <!-- 唯一 ref：避免与页面上其他 ECharts 实例冲突 -->
  <div ref="footprintChartRef" class="footprint-map"></div>
</template>

<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'
import * as echarts from 'echarts'
// 中国地图 GeoJSON（ECharts 5 不再内置地图数据，需自行引入）
import chinaJson from '@/assets/map/china.json'

/**
 * 足迹数据：
 * name   —— 省份名，必须与 china.json 里的地理名称一致（如"四川"、"北京"）
 * coords —— 图钉落点经纬度（建议填该省会城市坐标）
 */
const footprints = [
  { name: '北京', coords: [116.405285, 39.904989] },
  { name: '四川', coords: [103.834372, 30.057472] },
]

const footprintChartRef = ref(null)
let chart = null
let resizeObserver = null

// 只注册一次地图，防止热更新或多次挂载时重复注册
if (!echarts.getMap('china')) {
  echarts.registerMap('china', chinaJson)
}

const buildOption = () => ({
  backgroundColor: 'transparent',
  tooltip: {
    trigger: 'item',
    formatter: (params) =>
      params.seriesType === 'scatter'
        ? `打卡足迹：${params.name} 📍`
        : `到访过：${params.name}`,
  },
  geo: {
    map: 'china',
    roam: false, // 如需缩放拖拽改为 true
    scaleLimit: { min: 1, max: 4 },
    label: { show: false },
    itemStyle: {
      areaColor: '#e9eef5',      // 未去过：浅灰蓝
      borderColor: '#c3cfdd',
      borderWidth: 0.8,
    },
    emphasis: {
      label: { show: true, color: '#1f2d3d' },
      itemStyle: { areaColor: '#cfdcea' },
    },
    select: { disabled: true },
    // 去过的省份：红色高亮
    regions: footprints.map((item) => ({
      name: item.name,
      itemStyle: {
        areaColor: '#ff6b5e',
        borderColor: '#d93a2e',
        borderWidth: 1,
      },
      label: { show: true, color: '#7f1d1d', fontWeight: 600 },
    })),
  },
  series: [
    {
      name: '足迹图钉',
      type: 'scatter',
      coordinateSystem: 'geo',
      zlevel: 3,
      data: footprints.map((item) => ({ name: item.name, value: [...item.coords, 1] })),
      symbol: 'pin',
      symbolSize: 36,
      itemStyle: {
        color: '#d92b2b',
        shadowBlur: 10,
        shadowColor: 'rgba(0, 0, 0, 0.35)',
      },
      label: {
        show: true,
        formatter: '{b}',
        position: 'top',
        color: '#333',
        fontSize: 12,
      },
      emphasis: { scale: 1.15 },
    },
  ],
})

const initChart = () => {
  const el = footprintChartRef.value
  if (!el) return
  // getInstanceByDom 兜底：即使意外重复初始化，也复用同一实例
  chart = echarts.getInstanceByDom(el) ?? echarts.init(el)
  chart.setOption(buildOption())
}

onMounted(() => {
  initChart()
  // 监听容器尺寸变化（比 window.resize 更可靠，能覆盖侧边栏折叠等场景）
  resizeObserver = new ResizeObserver(() => chart?.resize())
  resizeObserver.observe(footprintChartRef.value)
})

onBeforeUnmount(() => {
  resizeObserver?.disconnect()
  resizeObserver = null
  chart?.dispose()
  chart = null
})
</script>

<style scoped>
/* 占满父级：父容器必须有明确高度（如 height: 100% / 100vh / 固定 px） */
.footprint-map {
  width: 100%;
  height: 100%;
  min-height: 360px; /* 兜底，防止父级高度塌陷时图表变形 */
}
</style>