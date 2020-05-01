<template>
  <div id="app">
    <input class="message" v-model="message" placeholder="String pentru compresie">
    <div :class="isEmpty? 'output-empty' : 'output'">Compression Tree</div>
    <Tree :message="message"></Tree>
  </div>
</template>

<script>
import Tree from './components/Tree'

export default {
  name: 'App',
  components:{
    Tree
  },
  computed:{
    isEmpty(){
      return this.message=='';
    }
  },
  data(){
    return{
      message:''
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.output-empty {
  padding: 7px;
  margin: 5px;
  color: rgb(255, 104, 34);
}

.output {
  padding: 7px;
  margin: 5px;
  color: rgb(27, 189, 27);
}

.node circle {
  fill: #fff;
  stroke: steelblue;
  stroke-width: 3px;
}

.node text {
  font: 12px sans-serif;
}

.link {
  fill: none;
  stroke: #ccc;
  stroke-width: 2px;
}

.DivToScroll {
  background-color: #f5f5f5;
  border: 1px solid #dddddd;
  border-radius: 4px 0 4px 0;
  color: #3b3c3e;
  font-size: 12px;
  font-weight: bold;
  left: -1px;
  padding: 10px 7px 5px;
}

.DivWithScroll {
  height: 120px;
  overflow: scroll;
  overflow-x: hidden;
}

.message {
  border: 1px solid #777;
  padding: 7px;
  margin: 5px;
}

.process-button {
  border: 1px solid #777;
  background-color: #555;
  color: white;
  padding: 15px;
}

.process-button:hover {
  background-color: #999;
  cursor: pointer;
}

</style>
