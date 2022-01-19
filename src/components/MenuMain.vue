<template>
  <div class="menu">
    <n-button class="btn" text @click="$emit('changeContent', 1)">
      <h1>ABOUT ME</h1>
    </n-button>
    <p>Former physics student and part-time Three.js fan</p>
    <div class="line"></div>

    <n-button text class="btn">
      <h1><a href="https://troisjs.github.io/">EXPLORE TroisJS</a></h1>
    </n-button>
    <p>A cool Three.js + Vue.js framework powering this website</p>
    <div class="line"></div>

    <n-button text class="btn" @click="$emit('changeContent', 2)">
      <h1>RESOURCES AND TOOLS</h1>
    </n-button>
    <p>Other frameworks, websites, and tools that helped me build this page</p>
    <div class="line"></div>
    <Icon size="48" color="white" >
      <WeatherRainShowersDay24Filled />
    </Icon>
  </div>
</template>

<script setup>
import { NButton} from 'naive-ui'
import { WeatherRainShowersDay24Filled } from '@vicons/fluent'
import { Icon } from '@vicons/utils'
const emits = defineEmits(['changeContent'])
</script>

<style scoped>
.line{
  height: 10vh;
  width: 2px;
  background-color: white;
  margin: 1vh auto;
}
.btn{
  --border-height: 3px;
}

</style>