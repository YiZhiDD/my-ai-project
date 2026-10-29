<template>
  <div style="padding: 20px;">
    <h2>1. 我的表单组件：</h2>
    <TravelForm @generate="(data) => alert('收到数据: ' + JSON.stringify(data))" />
    
    <hr style="margin: 40px 0;" />
    
    <h2>2. 我的地图组件：</h2>
    <ChinaMap />
  </div>
</template>

<script setup>
// 引入你写的两个组件
import TravelForm from './components/TravelForm.vue'
import ChinaMap from './components/ChinaMap.vue'
</script>