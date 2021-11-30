<template>
  <section id="loading-screen">
    <div id="loader"></div>
  </section>
  <Renderer
    ref="renderer"
    antialias
    :orbit-ctrl="{enableZoom: true}"
    resize="window"
    :pointer="{ onMove: onPointerMove }"
    shadow>
    <Camera ref="camera" :position="{x: 100, y: 70, z: 30}" :lookAt="{ x: 0, y: 0, z: 0}" :far="30000"/>
    <Scene ref="scene">
      <PointLight
        ref="mouse"
        :intensity="0.1"
        cast-shadow>
        <Sphere :radius="0.1" />
      </PointLight>
      <PointLight
        ref="main"
        :position="{ y: 100, z: 50 }"
        :intensity="brightness"
        cast-shadow>
      </PointLight>
      <AmbientLight />
      <PointLight :color="light1Color" :position="{ x: 50, y: 50, z: 50}" />
      <PointLight :color="light2Color" :position="{ x: -50, y: 50, z: 50}" />
      <PointLight :color="light3Color" :position="{ x: -50, y: -50, z: 50}" />
      <PointLight :color="light4Color" :position="{ x: 50, y: -50, z: 50}" />

      <FbxModel src="/assets/models/Dying.fbx"
        @load="onLoad" 
        :position="{y: 5, z: -10}"
        :scale="{ x: 0.1, y: 0.1, z: 0.1}"/>
      <!-- <GltfModel src="/assets/models/pony_cartoon/scene.gltf" 
      :position="{y: 5, z: -10}"
      /> -->
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
          :width="100"
          :depth="100"
          :position="{x:0, y: -1, z: 0}"
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
      <LiquidPlane
          ref="liquid"
          :width="WIDTH"
          :height="HEIGHT"
          :width-segments="512"
          :height-segments="512"
          :metalness="metalness"
          :roughness="roughness"
          :color="color"/>
    </Scene>
  </Renderer>
</template>

<script>
import * as THREE from 'three';
import CannonWorld from 'troisjs/src/components/physics/CannonWorld.js';
import LiquidPlane from 'troisjs/src/components/liquid/LiquidPlane.js';
import {Pane} from 'tweakpane';
import niceColors from 'nice-color-palettes';
export default {
  components:{
    CannonWorld,
    LiquidPlane
  },
  data(){
    //ball
    function coGen(n){
      var coords = []
      for (var i = 0; i <= n; i++){
        for (var j = -i; j <= i; j+=2){
          coords.push({x: j, y: 1, z: i * Math.sqrt(3)})
        }
      }
      return coords
    }
    var coords = coGen(13);
    var count = coords.length;
    const colors = new Array(count).fill().map(() => niceColors[20][Math.floor(THREE.MathUtils.randFloat(0, 5))]);
    var balls = [];
    for (var i = 0; i < count; i++){
      balls.push({position: coords[i], color: colors[i]})
    }
    //
    return{
      balls: balls,
      isInit: true,
      texturesProps: {
        repeat: { x: 5, y: 5 },
        wrapS: THREE.RepeatWrapping,
        wrapT: THREE.RepeatWrapping,
      },
      brightness: 1,
      metalness: 1,
      roughness: 0.2,
      light1Color: '#FFFF80',
      light2Color: '#DE3578',
      light3Color: '#FF4040',
      light4Color: '#0d25bb',
      color: '#ffffff',
      WIDTH: 30,
      HEIGHT: 30,
    }
  },
  methods:{
    onBeforeStep(){
      if(this.isInit){
        this.cue.userData.body.velocity.set(0, 0, 10);
        this.isInit = false;
      }
    },
    initCue(mesh){
      this.cue = mesh;
      mesh.userData.mass = 20;
    },
    initBalls(mesh){
      mesh.userData.mass = 1;
    },
    onLoad(object) {
      this.mixer = new THREE.AnimationMixer(object);
      const action = this.mixer.clipAction(object.animations[0]);
      action.play();
      object.traverse(function (child) {
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
     onPointerMove() {
      this.raycaster.setFromCamera(this.pointer.positionN, this.$refs.renderer.three.camera);
      this.raycaster.ray.intersectPlane(this.pointerPlane, this.pointerV3);
      const x = 2 * this.pointerV3.x / this.WIDTH;
      const y = 2 * this.pointerV3.y / this.HEIGHT;
      this.liquidEffect.addDrop(x, y, 0.025, 0.005);
    },
  },
  beforeCreate() {
    THREE.DefaultLoadingManager.onLoad = function () {
      console.log('Loading Complete!');
      document.getElementById("loading-screen").remove();
    }
  },
  mounted() {
    //scene core
    const renderer = this.$refs.renderer;
    const scene = this.$refs.scene.scene;

    //main light
    const main = this.$refs.main.light;
    const lightHelper = new THREE.PointLightHelper(main)
    scene.add(lightHelper);

    //orbit
    const orbitCtrl = renderer.three.cameraCtrl;
    orbitCtrl.target.set(0, 10, 0)

    //skybox
    let textureArray = [];
    let materialArray = [];
    let imageArray = [
    '/assets/skybox/tears_ft.jpg',
    '/assets/skybox/tears_bk.jpg',
    '/assets/skybox/tears_up.jpg',
    '/assets/skybox/tears_dn.jpg',
    '/assets/skybox/tears_rt.jpg',
    '/assets/skybox/tears_lf.jpg',];
    imageArray.forEach(element => textureArray.push(new THREE.TextureLoader().load(element)))
    textureArray.forEach(element => materialArray.push(new THREE.MeshBasicMaterial( { map: element })))
    for (let i = 0; i < 6; i++)
      materialArray[i].side = THREE.BackSide;
    let skyboxGeo = new THREE.BoxGeometry( 10000, 10000, 10000);
    let skybox = new THREE.Mesh( skyboxGeo, materialArray );
    scene.add( skybox );

    //tweakpane
    this.pane = new Pane();
    this.pane.addInput(this, 'brightness', { min: 0, max: 3 });
    
    //lantern
    const mouse = this.$refs.mouse.light;
    this.pointer = this.$refs.renderer.three.pointer
    const mouseV3 = this.pointer.positionV3;

    //liquid
    this.liquidEffect = this.$refs.liquid.liquidEffect;
    this.liquidEffect.addDrop(0, 0, 0.05, 0.05);

    this.raycaster = new THREE.Raycaster();
    this.pointerPlane = new THREE.Plane(new THREE.Vector3(0, 0, 1), 0);
    this.pointerV3 = new THREE.Vector3();



    //animation loop

    renderer.onBeforeRender(() => {
      mouse.position.copy(mouseV3);
    })
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