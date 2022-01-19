<template>
  <div class="menu" style="width: 500px"> 
    <h1>About Me</h1>
    <p>My name is Albert Chen. I'm a currently in the middle of my gap year before grad school and working as a software engineer intern in Shanghai, China.
    During my undergraduate years, I studied physics at Emory and Northwestern. 
    </p>
    <p>I enjoy web development, computer graphics, and computational physics. Visit my <a href="https://github.com/1AlbertChen"><u>GitHub</u></a> page and checkout my programming projects.</p>
    <p>This website is built with <a href="https://troisjs.github.io/"><u>TroisJS</u></a>, a Vue framework for Three.js. I contribute to this open-source framework in my spare time.</p>
    
    <h2>Technologies I enjoy using</h2>
    <div id="stack">
      <Icon size="24" color="#e44d26" >
        <Html5 />
      </Icon>
      <Icon size="24" color="#42a5f5" >
        <Css3 />
      </Icon>
      <Icon size="24" color="#ffca28" >
        <LogoJavascript />
      </Icon>
      <img src="https://img.icons8.com/color/48/000000/vue-js.png" />
      <Icon size="24" color="#00bad2" >
        <React />
      </Icon>
      <a href="https://threejs.org/">
        <img src="/assets/icons/threejs.jpg"/>
      </a>
      <img src="https://img.icons8.com/color/48/000000/python--v1.png" />
      <img src="https://img.icons8.com/color/48/000000/java-coffee-cup-logo--v1.png" />
      <a href="https://nodejs.org/">
        <img src="https://img.icons8.com/fluency/48/000000/node-js.png" />
      </a>
      <a href="https://www.mongodb.com/">
        <img src="https://img.icons8.com/color/48/000000/mongodb.png" />      
      </a>
      <a href="https://v3.nuxtjs.org/">
        <img src="https://img.icons8.com/external-tal-revivo-shadow-tal-revivo/24/000000/external-nuxt-js-a-free-and-open-source-web-application-framework-logo-shadow-tal-revivo.png"/>
      </a>
      <img src="https://img.icons8.com/color/48/000000/typescript.png"/>
    </div>
  </div>
</template>

<script setup>
import { Html5, Css3, React } from '@vicons/fa'
import { LogoJavascript } from '@vicons/ionicons5'
import { Icon } from '@vicons/utils'
</script>

<style scoped>
img{
  height: 24px;
}
#stack{
  justify-content: center;
  display: flex;
  gap: 10px;
}
</style>