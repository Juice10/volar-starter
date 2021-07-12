<script lang="ts">
import { defineComponent } from 'vue';
import HelloWorld from './components/HelloWorld.vue'
export default defineComponent({
  components: {
    HelloWorld
  },
  data() {
    return {
      selectedVoiceOverId: null,
    }
  },
})
</script>

<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <HelloWorld
    msg="Hello Vue 3 + TypeScript + Vite"
    :voice-over-id="selectedVoiceOverId"
    @select-voice-over-id="selectedVoiceOverId = $event"
  />
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>