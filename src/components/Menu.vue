<template>
  <n-button
    text
    @click="$emit('toggleSound')"
    class="absolute mute"
    v-if="soundMuted">
    <Icon size="40" :color="iconColor" >
      <LineHorizontal120Filled />
    </Icon>
  </n-button>

  <n-button
    text
    @click="$emit('toggleSound')"
    class="absolute mute"
    v-if="!soundMuted">
    <Icon size="40" :color="iconColor" >
      <WaveSine />
    </Icon>
  </n-button>

  <n-tooltip trigger="hover" width="300">
    <template #trigger>
      <n-button
        text
        style="left: 50%; top: 30px;"
        class="absolute">
        <Icon size="40" :color="iconColor" >
          <ConstructionRound />
        </Icon>
      </n-button>
    </template>
    This website is a work in progress. In the future, clicking the button here will switch the scene to the weather at your city.
  </n-tooltip>

  <n-button
    text
    @click="showMenu"
    style="top: 30px; left: 30px"
    class="absolute">
    <Icon size="40" :color="iconColor" >
      <MenuRound />
    </Icon>
  </n-button>

  <n-drawer
    v-model:show="menuShown"
    placement="bottom"
    style="backdrop-filter: blur(15px); background-color: rgba(255, 255, 255, 0.05); height: 100%;"
    content-style="display:flex; justify-content: center; align-items: center;">
    <n-button
      text
      @click="menuShown = false"
      style="top: 30px; right: 30px"
      class="absolute">
      <Icon size="40" color="white" >
        <CloseRound />
      </Icon>
    </n-button>

    <MenuMain v-if="menuContent == 0" @changeContent="onChangeContent"/>
    <MenuAbout v-if="menuContent == 1"/>
    <MenuResource v-if="menuContent == 2"/>

    <div id="contact" class="absolute">
      <a href="https://github.com/1AlbertChen">
        <Icon size="24" color="white" >
          <Github />
        </Icon>
      </a>

      <Icon size="24" color="#0072b1" >
        <Linkedin />
      </Icon>
      <a href="mailto:zhenrongchen2021@u.northwestern.edu">
        <Icon size="24" color="#f1cf3b" >
          <EmailRound />
        </Icon>
      </a>
      
    </div>
    <n-button
      text
      class="absolute btn"
      id="back"
      v-if="menuContent == 1 || menuContent == 2"
      @click="menuContent = 0">
      <p style="margin-bottom: 5px">Back to Menu</p>
    </n-button>
    
  </n-drawer>
</template>

<script setup>
import { computed, ref} from 'vue'
import { NButton, NDrawer, NTooltip } from 'naive-ui'
import { MenuRound, CloseRound, EmailRound, ConstructionRound} from '@vicons/material'
import { LineHorizontal120Filled } from '@vicons/fluent'
import { WaveSine} from '@vicons/tabler'
import { Github, Linkedin} from '@vicons/fa'
import { Icon } from '@vicons/utils'
import MenuAbout from './MenuAbout.vue'
import MenuMain from './MenuMain.vue'
import MenuResource from './MenuResource.vue'
const props = defineProps({
  vPosition: Number,
  soundMuted: Boolean
})
const emits = defineEmits(['toggleSound'])
const iconColor = computed(() => (props.vPosition > 15000) ? '#4d290b' : 'white')
const menuShown = ref(false)
const menuContent = ref(0)
const showMenu = () => {
  menuShown.value =  menuShown.value ? false : true
  menuContent.value = 0
}
const onChangeContent = (ev) => {
  menuContent.value = ev
}
</script>

<style scoped>
.mute{
  top: 30px;
  right: 30px
}
#contact{
  display: flex;
  gap: 10px;
  right: 50px;
  bottom: 30px;
}
#back{
  left: 50px;
  bottom: 30px;
}
.btn{
  --border-height: 2px;
}
</style>