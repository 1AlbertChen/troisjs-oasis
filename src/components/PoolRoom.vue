<template>
  <Renderer ref="rendererC" antialias :orbit-ctrl="{ enableDamping: true }" resize="window">
    <Camera :position="{ z: 10 }" />
    <Scene>
      <!-- <CannonWorld :gravity="{ x: 0, y: -9.82, z: 0}" @before-step="onBeforeStep">  -->
        <!-- <Sphere
          v-for="item in balls"
          :position="item.position"
          :radius="1"
          :widthSegments="30"
          :heightSegments="30"
          @created="initBalls"
          cast-shadow>
          <PhongMaterial :color="item.color" />
        </Sphere>
        <Sphere
          :position="{x: 0, y: 1, z: -20}"
          :radius="1"
          :widthSegments="30"
          :heightSegments="30"
          @created="initCue"
          cast-shadow>
          <PhongMaterial color="#ffffff" />
        </Sphere> --> 
      <!-- </CannonWorld> -->
    </Scene>
  </Renderer>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { Box, Camera, LambertMaterial, PointLight, Renderer, Scene } from 'troisjs'
const rendererC = ref()
const meshC = ref()
//balls
/* const RANKS = 13;
let coords = [];
for (let i = 0; i <= RANKS; i++){
  for (let j = -i; j <= i; j+=2){
    coords.push({x: j, y: 1, z: i * Math.sqrt(3)});
  }
} 
const colors = new Array(coords.length).fill().map(() => niceColors[20][Math.floor(THREE.MathUtils.randFloat(0, 5))]);
let balls = [];
for (var i = 0; i < coords.length; i++){
  balls.push({position: coords[i], color: colors[i]});
}
const initBalls = (mesh) => {mesh.userData.mass = 1} */


 //initialize cue
    /* onBeforeStep(){
      if(this.isInit){
        this.cue.userData.body.velocity.set(0, 0, 100);
        this.isInit = false;
      }
    }, */
    /* initCue(mesh){
      this.cue = mesh;
      mesh.userData.mass = 20;
    },
 */

onMounted(() => {
  const renderer = rendererC.value
  const mesh = meshC.value.mesh
  renderer.onBeforeRender(() => {
    mesh.rotation.x += 0.01
  })
})
</script>

<style>
body {
  margin: 0;
}
canvas {
  display: block;
}
</style>