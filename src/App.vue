<template>
  <div id="app">
    <p style="text-align: center">扫雷游戏怎么玩：鼠标左键点击，右键进行标注,
      游戏左方数字代表雷数，右方数字代表时间。</p>
    <HelloWorld/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
#app {
  margin-top: 60px;
}
</style>
