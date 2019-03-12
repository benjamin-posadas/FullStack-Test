<template>
  <div id="app">
    <GraphicContainer />
  </div>
</template>

<script>
  import GraphicContainer from './GraphicContainer';
  export default {
    name: 'app',
    components: {
      GraphicContainer
    },
    data () {
      return {}
    }
  }
</script>

<style>
  html,body{
    font-family: Verdana;
    margin: 0;
    height: 100%;
  }
  
  #app{
    height: 100%;
    background-color: #E0F7FA;
  }
</style>
