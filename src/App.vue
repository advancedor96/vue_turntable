<template>
  <div id="app">
    <TurnTable/>
  </div>
</template>

<script>
import TurnTable from './components/TurnTable'
export default {
  name: 'app',
  components: { TurnTable },
  data () {
    return {
    }
  },
  methods: {
    // hit () {
    //   let result = this.getRandomInt(1, 2)
    //   this.result = result
    // },
    // getRandomInt (min, max) {
    //   min = Math.ceil(min)
    //   max = Math.floor(max)
    //   return Math.floor(Math.random() * (max - min + 1)) + min
    // }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  // text-align: center;
  // color: #2c3e50;
  // margin-top: 60px;
}
.btn{
  width: 200px;
  height: 100px;
}
.result{
  font-size: 40px;
}
</style>
