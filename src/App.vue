<template>
  <section id="loading-screen">
    <div id="loader"></div>
  </section>
  <button @click="toggleSound">MUTED</button>
  <Renderer
    ref="renderer"
    orbit-ctrl
    resize="window"
    :pointer="{onMove: onPointerMove}"
    >
    <Camera ref="camera" :position="{x: 20, y: 30, z: 0}" />
    <Scene ref="scene" :background="cubeTexture">
    <!-- <Scene ref="scene"> -->
      <PointLight
        ref="mouse"
        :intensity="0"
        cast-shadow>
        <Sphere :radius="0.1" />
      </PointLight>
      <PointLight
        ref="main"
        :position="{ y: 40 }"
        :intensity="brightness"
        cast-shadow/>
      <PointLight
        ref="flash"
        :position="{x: 200,   y: 300, z: 100}"
        :intensity="30"
        :decay="1.7"  
        :distance="500"
        color="#062d89"
        />

      <FbxModel src="/assets/models/Dying.fbx"
        @load="onLoad" 
        :position="{y: 5, z: -10}"
        :scale="{ x: 0.1, y: 0.1, z: 0.1}"/>

      <Plane
      v-for="i in 25"
      :ref="`mesh${i}`"
      :width="500"
      :height="500"
      :position="{x: Math.random()*800-400, y: 500, z: Math.random()*500-450 }"
      :rotation="{x: 1.16, y: -0.12, z: Math.random()*360}"
      >
        <LambertMaterial
        :props="{transparent: true, opacity: 0.6, depthWrite:false}">
          <Texture src="/assets/textures/smoke.png" />
        </LambertMaterial>
      </Plane>

      <LiquidPlane
        ref="liquid"
        :width="width"
        :height="height"
        :material-props="materialProps"
        :rotation="{ x: -Math.PI / 2 }"/>


      <CannonWorld :gravity="{ x: 0, y: -9.82, z: 0}" @before-step="onBeforeStep"> 
        <Sphere
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
        </Sphere>
        <Box
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
        </Box>  
      </CannonWorld>
    </Scene>
  </Renderer>

  <audio id="mySong">
    <source src="/assets/sounds/rain.wav" type="audio/wav"/>
  </audio>
</template>

<script>
//TODO:
/*
Add UI
Add sound player*/
import * as THREE from 'three';
import CannonWorld from 'troisjs/src/components/physics/CannonWorld.js';
import LiquidPlane from '@troisjs/components/src/liquid/LiquidPlane.js'
import {Pane} from 'tweakpane';
import niceColors from 'nice-color-palettes';
export default {
  components:{
    CannonWorld,
    LiquidPlane,
  },
  setup(){
    //skybox
    let imageArray = [
    '/assets/skybox/battery_ft.jpg',
    '/assets/skybox/battery_bk.jpg',
    '/assets/skybox/battery_up.jpg',
    '/assets/skybox/battery_dn.jpg',
    '/assets/skybox/battery_rt.jpg',
    '/assets/skybox/battery_lf.jpg',];

    const cubeTexture = new THREE.CubeTextureLoader().load(imageArray)
    
    //balls
    const RANKS = 13
    let coords = []
    for (let i = 0; i <= RANKS; i++){
      for (let j = -i; j <= i; j+=2){
        coords.push({x: j, y: 1, z: i * Math.sqrt(3)})
      }
    }
    const colors = new Array(coords.length).fill().map(() => niceColors[20][Math.floor(THREE.MathUtils.randFloat(0, 5))]);
    let balls = [];
    for (var i = 0; i < coords.length; i++){
      balls.push({position: coords[i], color: colors[i]})
    }
    
    const initBalls = (mesh) => {mesh.userData.mass = 1}

    //Loading Manager
    THREE.DefaultLoadingManager.onLoad =  () => {
      console.log('Loading Complete!');
      document.getElementById("loading-screen").remove();
    }

    //toggleSound
    const toggleSound = () =>{
      let song = document.getElementById("mySong")
      song.muted = song.muted ? false : true
    }
    

    return{
      toggleSound,
      cubeTexture,
      balls,
      initBalls,
      texturesProps: {
        repeat: { x: 5, y: 5 },
        wrapS: THREE.RepeatWrapping,
        wrapT: THREE.RepeatWrapping,
      },
      width: 100,
      height: 100,
    }
  },
  data(){
    return{
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
      //init
      isInit: true,
      //light
      brightness: 0.7,
      //liquid
      color: 0xffffff,
      metalness: 0.7,
      roughness: 0.2,
      //rain
      rainCount: 15000
    }
  },
  mounted() {
    //tweakpane
    this.pane = new Pane();
    this.pane.addInput(this, 'brightness', { min: 0, max: 3 });
    this.pane.addInput(this.materialProps, 'metalness', { min: 0, max: 1 });
    this.pane.addInput(this.materialProps, 'roughness', { min: 0, max: 1 });

    //scene core
    const renderer = this.$refs.renderer;
    const camera = this.$refs.camera.camera;
    const scene = this.$refs.scene.scene;
    const flash = this.$refs.flash.light;

    //main light helper
    const main = this.$refs.main.light;
    const lightHelper = new THREE.PointLightHelper(main)
    scene.add(lightHelper);

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
    let rainVert = new Float32Array(this.rainCount*3)
    for(let i = 0; i < this.rainCount; i++) {
      rainVert[i*3] = Math.random() * 400 -200,
      rainVert[i*3 + 1] =  Math.random() * 250,
      rainVert[i*3 + 2] =  Math.random() * 400 - 200
    }
    rainGeo.setAttribute('position', new THREE.BufferAttribute( rainVert, 3 ));
    const rain = new THREE.Points(rainGeo, rainMaterial);
    scene.add(rain)
 
    //liquid
    this.liquidEffect = this.$refs.liquid.liquidEffect;
    this.liquidEffect.addDrop(0, 0, 0.05, 0.05);

    this.raycaster = new THREE.Raycaster();
    this.pointerPlane = new THREE.Plane(new THREE.Vector3(0, 1, 0), 0);
    this.pointerV3 = new THREE.Vector3();

    var mySong = document.getElementById("mySong")
    document.onclick = () => mySong.play()


    //ANIMATION LOOP
    renderer.onBeforeRender(() => {
      //mouse
      mouse.position.copy(mouseV3);
      //rain
      
      for(let i = 0; i < this.rainCount; i++){
        rainVert[i*3 + 1] -= 2
        if(rainVert[i*3 + 1] < 0){
          rainVert[i*3 + 1] = 250
        }
      }
      rainGeo.setAttribute('position', new THREE.BufferAttribute( rainVert, 3 ));
      
      //cloud
      for (let i = 1; i <= 25; i++) {
        let mesh = this.$refs['mesh'+i].mesh;
        mesh.rotation.z -= 0.003;
      }
      //thunder
      if(Math.random() > 0.96 || flash.power > 100) {
        if(flash.power < 100) 
          flash.position.set(
            Math.random()*400,
            300 + Math.random() *200,
            100
          );
        flash.power = 50 + Math.random() * 300;
      }
    })
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

    //initialize cue
    onBeforeStep(){
      if(this.isInit){
        this.cue.userData.body.velocity.set(0, 0, 100);
        this.isInit = false;
      }
    },
    initCue(mesh){
      this.cue = mesh;
      mesh.userData.mass = 20;
    },

    //model
    onLoad(object) {
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
    },
  },
  unmounted() {
    this.pane.dispose();
  },
}
</script>

<style>
#loading-screen {
	position: absolute;
	z-index: 2;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	background-color: #000000;
	opacity: 1;
 	transition: 1s opacity;
}

#loading-screen.fade-out {
    opacity: 0;
}

#loader {
    display: block;
    position: relative;
    left: 50%;
    top: 50%;
    width: 150px;
    height: 150px;
    margin: -75px 0 0 -75px;
    border-radius: 50%;
    border: 3px solid transparent;
    border-top-color: #9370DB;
    -webkit-animation: spin 2s linear infinite;
    animation: spin 2s linear infinite;
}
#loader:before {
    content: "";
    position: absolute;
    top: 5px;
    left: 5px;
    right: 5px;
    bottom: 5px;
    border-radius: 50%;
    border: 3px solid transparent;
    border-top-color: #BA55D3;
    -webkit-animation: spin 3s linear infinite;
    animation: spin 3s linear infinite;
}
#loader:after {
    content: "";
    position: absolute;
    top: 15px;
    left: 15px;
    right: 15px;
    bottom: 15px;
    border-radius: 50%;
    border: 3px solid transparent;
    border-top-color: #FF00FF;
    -webkit-animation: spin 1.5s linear infinite;
    animation: spin 1.5s linear infinite;
}
@-webkit-keyframes spin {
    0%   {
        -webkit-transform: rotate(0deg);
        -ms-transform: rotate(0deg);
        transform: rotate(0deg);
    }
    100% {
        -webkit-transform: rotate(360deg);
        -ms-transform: rotate(360deg);
        transform: rotate(360deg);
    }
}
@keyframes spin {
    0%   {
        -webkit-transform: rotate(0deg);
        -ms-transform: rotate(0deg);
        transform: rotate(0deg);
    }
    100% {
        -webkit-transform: rotate(360deg);
        -ms-transform: rotate(360deg);
        transform: rotate(360deg);
    }
}

</style>