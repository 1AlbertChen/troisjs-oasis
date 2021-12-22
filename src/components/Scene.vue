<template>
  <transition name="fade">
    <Loader v-if="loaded" :progress="percent" @initPage="initPage"/>
  </transition>
  <n-switch @click="toggleSound" style="display: absolute;">
    <template #checked>Play</template>
    <template #unchecked>Mute</template>
  </n-switch>
  <button @click="rainUnder += 2000">LESS</button>
    <button @click="rainUnder -= 2000">MOAR</button>
  <Renderer
    ref="renderer"
    orbit-ctrl
    resize="window">
    <!-- :pointer="{onMove: onPointerMove}" -->
    <Camera ref="camera" :position="{x: 0, y: 100, z: 0}" :far="10000"/>
    <Scene ref="scene">
      <PointLight
        ref="mouse"
        :intensity="0.5"
        cast-shadow>
        <Sphere :radius="0.1"/>
      </PointLight>

      <PointLight
        ref="main"
        :color="color"
        :position="{ y: 100 }"
        :intensity="brightnessPt"
        cast-shadow/>

      <SpotLight
        color="#555555"
        :position="{ y: 250 }"
        :distance="500"
        :angle="angle"
        :intensity="brightnessDr"
        :target="{ y: 500 }"
      />
      <PointLight
        ref="flash"
        :position="{x: 200, y: 300, z: 100}"
        :intensity="30"
        :decay="1.7"  
        :distance="500"
        color="#062d89"
        />
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

    <!-- <LiquidPlane
        ref="liquid"
        :width="width"
        :height="height"
        :material-props="materialProps"
        :rotation="{ x: -Math.PI / 2 }"/> -->

      <Plane
        :rotation="{ x: -Math.PI / 2}"
        :width="1000"
        :height="1000"
        :widthSegments="500"
        :heightSegments="500"
        :position="{x: 0, y: -5, z: 0}"
        receive-shadow>
        <StandardMaterial :props="{ displacementScale: 20 }">
          <Texture :props="texturesProps" src="/assets/textures/beachsand.jpg" />
          <Texture :props="texturesProps" src="/assets/textures/noiseTexture1.png" name="displacementMap" />
        </StandardMaterial>
      </Plane> 

      <CannonWorld :gravity="{ x: 0, y: -9.82, z: 0}" @before-step="onBeforeStep"> 
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
        <!-- <Box
          :width="20"
          :depth="20"
          :position="{x:0, y: -10, z: 0}"
          receive-shadow>
           <StandardMaterial :props="{ displacementScale: 0.2 }">
            <Texture :props="texturesProps" src="/assets/textures/Wood_Tiles_002_basecolor.jpg" />
            <Texture :props="texturesProps" src="/assets/textures/Wood_Tiles_002_height.png" name="displacementMap" />
            <Texture :props="texturesProps" src="/assets/textures/Wood_Tiles_002_normal.jpg" name="normalMap" />
            <Texture :props="texturesProps" src="/assets/textures/Wood_Tiles_002_roughness.jpg" name="roughnessMap" />
            <Texture :props="texturesProps" src="/assets/textures/Wood_Tiles_002_ambientOcclusion.jpg" name="aoMap" />
          </StandardMaterial> 
        </Box>  --> 
      </CannonWorld>
    </Scene>
  </Renderer>

  <audio id="rainSound" loop>
    <source src="/assets/sounds/rain.wav" type="audio/wav"/>
  </audio>
</template>

<script>
import * as THREE from 'three';
import {ref} from 'vue';
import CannonWorld from 'troisjs/src/components/physics/CannonWorld.js';
import LiquidPlane from '@troisjs/components/src/liquid/LiquidPlane.js'
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
    return{
      imageArray,
      loaded,
      percent,
      //balls,
      //initBalls,
      toggleSound,
      initPage,
      audioFade
    }
  },
  data(){
    return{
      //height: 40,
      angle: Math.PI,
      color: '#071021',
      //cue
      isInit: true,
      //light
      brightnessPt: 3,
      brightnessDr: 5,
      //liquid
      width: 100,
      height: 100,
      materialProps: {
        color: 0xffffff,
        metalness: 0.2,
        roughness: 0,
        thickness: 4,
        transmission: 1,
        displacementScale: 20,
        envMap: this.cubeTexture,
        envMapIntensity: 1
      },
      //wood
      texturesProps: {
        repeat: { x: 5, y: 5 },
        wrapS: THREE.RepeatWrapping,
        wrapT: THREE.RepeatWrapping,
      },
      //rain
      rainVert: [],
      rainCount: 10000,
      rainUnder: 0
    }
  },
  mounted() {
    //tweakpane
    this.pane = new Pane();
    this.pane.addInput(this, 'brightnessPt', { min: 0, max: 3 });
    this.pane.addInput(this, 'brightnessDr', { min: 0, max: 5 });
    this.pane.addInput(this, 'angle', { min: 0, max: Math.PI/2 });
    this.pane.addInput(this, 'color');

    //scene core
    const renderer = this.$refs.renderer;
    const scene = this.$refs.scene.scene;
    const flash = this.$refs.flash.light;
    scene.fog = new THREE.Fog(this.color, 100, 800)
    
    //main light helper
    const main = this.$refs.main.light;
    const lightHelper = new THREE.PointLightHelper(main)
    scene.add(lightHelper);
    
    //skybox
    let imageArray = [
    '/assets/skybox/blizzard_ft.jpg',
    '/assets/skybox/blizzard_bk.jpg',
    '/assets/skybox/blizzard_up.jpg',
    '/assets/skybox/blizzard_dn.jpg',
    '/assets/skybox/blizzard_rt.jpg',
    '/assets/skybox/blizzard_lf.jpg',];
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

    //rain
    const rainMaterial = new THREE.PointsMaterial({
      color: 0xaaaaaa,
      size: 0.1,
      transparent: true
    }); 

    let rainGeo = new THREE.BufferGeometry();
    this.rainVert = new Float32Array(this.rainCount*3)
    for(let i = 0; i < this.rainCount; i++) {
      this.rainVert[i*3] = Math.random() * 400 -200,
      this.rainVert[i*3 + 1] =  Math.random() * 250,
      this.rainVert[i*3 + 2] =  Math.random() * 400 - 200
    }
    rainGeo.setAttribute('position', new THREE.BufferAttribute( this.rainVert, 3 ));
    const rain = new THREE.Points(rainGeo, rainMaterial);
    scene.add(rain)
 
    //liquid
   /*this.liquidEffect = this.$refs.liquid.liquidEffect;
    this.liquidEffect.addDrop(0, 0, 0.05, 0.05);

    this.raycaster = new THREE.Raycaster();
    this.pointerPlane = new THREE.Plane(new THREE.Vector3(0, 1, 0), 0);
    this.pointerV3 = new THREE.Vector3(); */

    //ANIMATION LOOP
    renderer.onBeforeRender(() => {
      //mouse
      mouse.position.copy(mouseV3);
      //rain
      for(let i = 0; i < this.rainCount; i++){
        if(this.rainVert[i*3 + 1] < 0 && this.rainVert[i*3 + 1] > -10){
          this.rainVert[i*3 + 1] = 250
        }
        this.rainVert[i*3 + 1] -= 2
        
        
      }
      rainGeo.setAttribute('position', new THREE.BufferAttribute( this.rainVert, 3 ));
      
      //cloud
      for (let i = 1; i <= 25; i++) {
        let mesh = this.$refs['mesh'+i].mesh;
        mesh.rotation.z -= 0.003;
      }
      //Fog
      this.$refs.scene.scene.fog.color.set(this.color)
      //thunder
      if(Math.random() > 0.96 || flash.power > 100) {
        if(flash.power < 100) 
          flash.position.set(
            Math.random()*400,
            300 + Math.random() *200,
            100);
        flash.power = 50 + Math.random() * 300;
      }
    })
  },

  watch:{
    rainUnder(val, old){
      if(val > old){
        const lessRain = val - old
        for(let i = 0; i < lessRain; i++){
          this.rainVert[(this.rainUnder - lessRain + i )*3 + 1] = -50 //banished to the shadow realm
        }
      }
      else{
        const moreRain = old - val
        for(let i = 0; i < moreRain; i++){
          this.rainVert[(this.rainUnder + i)*3 + 1] = Math.random() * 250
        }
      }

    }
  },
 
  methods:{
    //liquid
    /* onPointerMove() {
      this.raycaster.setFromCamera(this.pointer.positionN, this.$refs.renderer.three.camera);
      this.raycaster.ray.intersectPlane(this.pointerPlane, this.pointerV3);
      const x = 2 * this.pointerV3.x / this.width;
      const y = 2 * -this.pointerV3.z / this.height;
      this.liquidEffect.addDrop(x, y, 0.025, 0.005);
    }, */

    //initialize cue
    onBeforeStep(){
      /* if(this.isInit){
        this.cue.userData.body.velocity.set(0, 0, 100);
        this.isInit = false;
      } */
    },
    /* initCue(mesh){
      this.cue = mesh;
      mesh.userData.mass = 20;
    },
 */
    //model
    /* onLoad(object) {
      this.mixer = new THREE.AnimationMixer(object);
      const action = this.mixer.clipAction(object.animations[0]);
      action.play();
      object.traverse((child) => {
        if (child.isMesh) {
          child.castShadow = true;
          child.receiveShadow = true;
        }
      });
      this.clock = new THREE.Clock();
      this.$refs.renderer.onBeforeRender(this.updateMixer);
    },
    updateMixer() {
      this.mixer.update(this.clock.getDelta());
    }, */
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