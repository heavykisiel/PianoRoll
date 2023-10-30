<template>
  <TopBar></TopBar>
    <PianoRoll></PianoRoll>

 

  
</template>

<script>
import TopBar from './components/TopBar.vue'
import PianoRoll from './components/PianoRoll.vue';



export default {
  name: 'App',
  components: {
    TopBar,
    PianoRoll,
}
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: 'Mulish', sans-serif;
  cursor: default;
  background-color: #F0F0F0;
  color: #1C1C1A;
  height: 100vh; /* Use viewport height instead of 100% to occupy full screen height */
  text-align: center;
  overflow: auto;
  display: flex;
  flex-direction: column;
  position: relative;
}
</style>
