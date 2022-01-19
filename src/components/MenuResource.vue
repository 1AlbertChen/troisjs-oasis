<template>
  <div class="menu">
    <h1>Resources and Tools</h1>
    <div id="tools">
      <n-tooltip
        width="300"
        v-for="item in icons"
        trigger="hover">
        <template #trigger>
          <a :href="item.link">
            <img :src="'/assets/icons/'+ item.name">
          </a> 
        </template>
        {{item.message}}
      </n-tooltip>
    </div>
  </div>
</template>

<script setup>
import { NTooltip } from 'naive-ui'
const icons = [
  {
    name: "vite.svg",
    message: "Vite.js. Vite means fast in French, and this thing is insanely fast!",
    link: "https://vitejs.dev/",
  },
  {
    name: "naive.svg",
    message: "NaiveUI, a Vue 3 component library. Too young too simple. Sometimes naive.",
    link: "https://www.naiveui.com/",
  },
  {
    name: "canva.png",
    message: "Canva. I only used it for the color palettes. But you can do more.",
    link: "https://www.canva.com/",
  },
  {
    name: "icons8.png",
    message: "Icons8. They might sue me if I don't put a link here.",
    link: "https://icons8.com/",
  },
  {
    name: "sketchfab.png",
    message: "Sketchfab, put a 3D model of a tank in a lake for the memes.",
    link: "https://sketchfab.com/",
  },
  {
    name: "blender.png",
    message: "Download crappy 3D models on Sketchfab, and make them less crappy in Blender.",
    link: "https://www.blender.org/",
  },
]
</script>

<style scoped>
img{
  height: 48px;
}
#tools{
  margin-top: 10vh;
  width: 50vw;
  height: 50vh;
  display: flex;
  flex-wrap: wrap;
  gap: 30px;
  justify-content: space-around;
  align-items: center;
}
</style>