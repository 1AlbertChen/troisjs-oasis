<template>
  <!-- <transition name="fade">
    <Loader v-if="loaded" :progress="percent" @initPage="initPage"/>
  </transition> -->
  <n-switch @click="toggleSound" style="position: absolute;left: 200px;">
    <template #checked>Play</template>
    <template #unchecked>Mute</template>
  </n-switch>
  <Renderer
    ref="renderer"
    antialias
    :pointer="{onMove: onPointerMove}"
    @wheel="e => loaded && onScroll(e)"
    resize="window">
    <Camera
      ref="camera"
      :position="{x: 100, y: 100, z: 100}"
      :far="10000"
      :lookAt="{x: 0, y: 0, z: 0}" />
    <Scene ref="scene">
      <PointLight
        ref="mouse"
        :intensity="0"
        cast-shadow>
        <Sphere :radius="0.1"/>
      </PointLight>
      
      <HemisphereLight
        :color="skycolor"
        :groundColor="groundcolor"
        :intensity="brightnessPt"
        cast-shadow/>

      <SpotLight
        color="#555555"
        :angle="angle"
        :distance="500"
        :intensity="brightnessDr"
        :position="{ y: 250 }"
        :target="{y: 500}"
      />
      <!-- <PointLight
        ref="flash"
        :position="{x: 200, y: 300, z: 100}"
        :intensity="30"
        :decay="1.7"  
        :distance="500"
        color="#062d89"
        /> -->
      <GltfModel  src="/assets/models/plants/tree.glb"
        :position="{x: 40}" :scale="{x: 3, y: 3, z: 3}" />
      <GltfModel  src="/assets/models/plants/tree2.glb"
        :position="{x: -40}" :scale="{x: 3, y: 3, z: 3}"/>
      <GltfModel  src="/assets/models/plants/grass/scene.gltf"
        :position="{x: 20}" :scale="{x: 0.5, y: 0.5, z: 0.5}"/>
      <GltfModel  src="/assets/models/plants/agave.glb"
        :position="{x: 20}" :scale="{x: 0.5, y: 0.5, z: 0.5}"/>
      <GltfModel src="/assets/models/plants/cactus-ball.glb"
        :position="{x: 0}" :scale="{x: 0.5, y: 0.5, z: 0.5}"/>
      <GltfModel  src="/assets/models/plants/cactus.glb"
        :position="{z: -60}" :scale="{x: 0.1, y: 0.1, z: 0.1}"/>
      <GltfModel  src="/assets/models/plants/bush.glb"
        :position="{z: 30}" :scale="{x: 10, y: 10, z: 10}"/>

      <!-- cloud -->
      <Plane
      v-for="i in 25"
      :ref="`mesh${i}`"
      :width="500"
      :height="500"
      :position="{x: Math.random()*800-400, y: 500, z: Math.random()*500-450 }"
      :rotation="{x: 1.16, y: -0.12, z: Math.random()*360}"
      >
        <StandardMaterial
        :props="{transparent: true, opacity: 0.6, depthWrite:false}">
          <Texture src="/assets/textures/smoke.png" />
        </StandardMaterial>
      </Plane>

    <LiquidPlane
      ref="liquid"
      :width="width"
      :height="height"
      :material-props="materialProps"
      :rotation="{ x: -Math.PI / 2 }"/>
      <Plane
        :rotation="{ x: -Math.PI / 2}"
        :width="800"
        :height="800"
        :widthSegments="64"
        :heightSegments="64"
        :position="{x: 0, y: elevation, z: 0}"
        receive-shadow>
        <StandardMaterial :props="{ displacementScale: scale }">
          <Texture src="/assets/textures/sand.jfif" />
          <Texture src="/assets/textures/Background.png" name="displacementMap" />
        </StandardMaterial>
      </Plane> 

      
    </Scene>
  </Renderer>

  <audio id="rainSound" loop>
    <source src="/assets/sounds/thunderstorm.wav" type="audio/wav"/>
  </audio>
</template>

<script>
import * as THREE from 'three';
import {ref} from 'vue';
import CannonWorld from 'troisjs/src/components/physics/CannonWorld.js';
import LiquidPlane from '@troisjs/components/src/liquid/LiquidPlane.js';
import {Pane} from 'tweakpane';
/* import niceColors from 'nice-color-palettes'; */
import Loader from './Loader.vue';
import {NSwitch} from 'naive-ui'
export default {
  components:{
    CannonWorld,
    LiquidPlane,
    Loader,
    NSwitch
  },
  setup(){
    //Loading Manager
    const percent = ref(0)
    THREE.DefaultLoadingManager.onProgress = function (url, itemsLoaded, itemsTotal){
      percent.value = (itemsLoaded / itemsTotal * 100);
    }

    //initPage
    const loaded = ref(true)
    const initPage = () =>{
      loaded.value = false
      const audio = document.getElementById("rainSound")
      audio.play()
      audio.volume = 0
      audioFade()
    }
    //scroll control
    const vPosition = ref(0)
    const onScroll = ev => {
      const delY = ev.deltaY
      vPosition.value += delY
      vPosition.value = Math.min(Math.max(0, vPosition.value), 1000);
      console.log(vPosition.value)
    }
    //sound control
    const audioFade = () =>{
      const myAudio = document.getElementById("rainSound");
      let timer;
      if (myAudio.volume < 0.98) {
        myAudio.volume += .01;
        timer = setTimeout(audioFade, 200);
      }
    }

    const toggleSound = () =>{
      const rainSound = document.getElementById("rainSound")
      rainSound.muted = rainSound.muted ? false : true
    }

    const imageArray = Array(6).fill('/assets/skybox/sky.png')
    const cubeTexture = new THREE.CubeTextureLoader().load(imageArray)

   
    return{
      vPosition,
      onScroll,
      loaded,
      percent,
      cubeTexture,
      imageArray,
      toggleSound,
      initPage,
      audioFade,
    }
  },
  data(){
    return{
      //ground
      elevation: -12,
      scale: 30,
      //lights
      skycolor:'#010a1a',
        //'#CCC7B0',
      groundcolor:'#071021',
        //'#ffffff',
      brightnessPt: 1,
      brightnessDr: 5,
      angle: Math.PI/2,
      //liquid
      width: 150,
      height: 150,
      materialProps: {
        color: 0xffffff,
        metalness: 0.15,
        roughness: 0,
        thickness: 1,
        transmission: 1,
        displacementScale: 20,
        reflectivity: 0.5,
        envMap: this.cubeTexture,
        envMapIntensity: 0.5
      },
      //rain
      rainVert: [],
      rainCount: 5000,
      rainUnder: 0,
    }
  },
  mounted() {
    (function(){var script=document.createElement('script');script.onload=function(){var stats=new Stats();document.body.appendChild(stats.dom);requestAnimationFrame(function loop(){stats.update();requestAnimationFrame(loop)});};script.src='//mrdoob.github.io/stats.js/build/stats.min.js';document.head.appendChild(script);})()
    //tweakpane
    this.pane = new Pane();
    this.pane.addInput(this, 'brightnessPt', { min: 0, max: 3 });
    this.pane.addInput(this, 'brightnessDr', { min: 0, max: 5 });
    this.pane.addInput(this, 'angle', { min: 0, max: Math.PI });
    this.pane.addInput(this, 'skycolor');
    this.pane.addInput(this, 'groundcolor');
    this.pane.addInput(this.materialProps, 'metalness', { step: 0.05, min: 0, max: 1 })
    this.pane.addInput(this.materialProps, 'reflectivity', { step: 0.05, min: 0, max: 1 })
    this.pane.addInput(this.materialProps, 'roughness', { step: 0.05, min: 0, max: 1 })
    this.pane.addInput(this.materialProps, 'thickness', { step: 0.05, min: 0, max: 10 })
    this.pane.addInput(this.materialProps, 'transmission', { step: 0.05, min: 0, max: 1 })
    this.pane.addInput(this.materialProps, 'displacementScale', { step: 0.5, min: 0.5, max: 50 })
    this.pane.addInput(this.materialProps, 'envMapIntensity', { step: 0.05, min: 0, max: 1 })
    this.pane.addInput(this, 'width', { min: 0, max: 1000 })
    this.pane.addInput(this, 'height', { min: 0, max: 1000 })
    /* this.pane.addInput(this, 'scale', { min: 0, max: 100 });
    this.pane.addInput(this, 'elevation', { min: -20, max: 0 }); */
    
    //scene core ***
    const renderer = this.$refs.renderer
    const scene = this.$refs.scene.scene
    const camera = this.$refs.camera.cameras
    scene.fog = new THREE.Fog(this.color, 1, 800)

    //skybox ***
    let texture = []
    let material = []
    this.imageArray.forEach((el) => texture.push(new THREE.TextureLoader().load(el)))
    texture.forEach((el) => material.push(new THREE.MeshStandardMaterial( { map: el })))
    for (let i = 0; i < 6; i++)
      material[i].side = THREE.BackSide;
    let skyboxGeo = new THREE.BoxGeometry( 10000, 10000, 10000);
    let skybox = new THREE.Mesh( skyboxGeo, material );
    scene.add( skybox );

    //mouse lantern
    const mouse = this.$refs.mouse.light;
    this.pointer = renderer.three.pointer
    const mouseV3 = this.pointer.positionV3;

    //rain ***
    const rainMaterial = new THREE.PointsMaterial({
      color: 0xaaaaaa,
      size: 0.1,
      transparent: true
    }); 

    const rainGeo = new THREE.BufferGeometry();
    let rainVert = new Float32Array(this.rainCount*3)
    for(let i = 0; i < this.rainCount; i++) {
      rainVert[i*3] = Math.random() * 400 -200,
      rainVert[i*3 + 1] =  Math.random() * 250,
      rainVert[i*3 + 2] =  Math.random() * 400 - 200
    }
    rainGeo.setAttribute('position', new THREE.BufferAttribute( rainVert, 3 ));
    this.rain = new THREE.Points(rainGeo, rainMaterial);
    scene.add(this.rain)
 
    //liquid
    this.liquidEffect = this.$refs.liquid.liquidEffect;
    this.liquidEffect.addDrop(0, 0, 0.05, 0.05);

    this.raycaster = new THREE.Raycaster();
    this.pointerPlane = new THREE.Plane(new THREE.Vector3(0, 1, 0), 0);
    this.pointerV3 = new THREE.Vector3();

   //***** */ const flash = this.$refs.flash.light;
    //ANIMATION LOOP
    renderer.onBeforeRender(() => {
      //mouse
      mouse.position.copy(mouseV3);
      //rain
      const positions = this.rain.geometry.attributes.position.array
      for(let i = 0; i < this.rainCount; i++ ){
        if (positions[ 3 * i + 1] > 0){
          positions[ 3 * i + 1] -= 2
        }
        else if (positions[ 3 * i + 1] > -10){
          positions[ 3 * i + 1] = 250
        }
      }
      this.rain.geometry.attributes.position.needsUpdate = true;

      //cloud
      for (let i = 1; i <= 25; i++) {
        let mesh = this.$refs['mesh'+i].mesh;
        mesh.rotation.z -= 0.003;
      }
      //Fog
      const fog = this.$refs.scene.scene.fog
      fog.color.set(this.skycolor)
      //thunder
      /* if(Math.random() > 0.98 || flash.power > 100) {
        if(flash.power < 100) 
          flash.position.set(
            Math.random()*400,
            300 + Math.random() *200,
            100);
        flash.power = 50 + Math.random() * 300;
      } */
    })
  },

  watch:{
    rainUnder(val, old){
      const positions = this.rain.geometry.attributes.position.array
      if(val > old){
        const lessRain = val - old
        for(let i = 0; i < lessRain; i++){
          positions[ 3 * (this.rainUnder - lessRain + i) + 1] = -50
        }
      }
      else{
        const moreRain = old - val
        for(let i = 0; i < moreRain; i++){
          positions[ 3 * (this.rainUnder  + i) + 1] = Math.random() * 250
       }
      }
      this.rain.geometry.attributes.position.needsUpdate = true;
    }
  },
 
  methods:{
    //liquid
    onPointerMove() {
      this.raycaster.setFromCamera(this.pointer.positionN, this.$refs.renderer.three.camera);
      this.raycaster.ray.intersectPlane(this.pointerPlane, this.pointerV3);
      const x = 2 * this.pointerV3.x / this.width;
      const y = 2 * -this.pointerV3.z / this.height;
      this.liquidEffect.addDrop(x, y, 0.025, 0.005);
    },
  },
  
}
</script>

<style scoped>
.fade-leave-active {
  transition: opacity 2s ease;
}
.fade-leave-to {
  opacity: 0;
}
</style>