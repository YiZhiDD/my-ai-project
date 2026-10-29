<template>
  <div>
    <!-- 引入了你新写的设置模块 -->
    <Settings />
  </div>
</template>

<script setup>
// 引入刚刚新建的 Settings 组件
import Settings from './components/Settings.vue'
</script>

<style>
/* 这里的样式先留空，由各个组件自己管理 */
</style>