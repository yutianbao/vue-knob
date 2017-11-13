<template>
    <knob v-model="knobValue" @changeKnob="knobChange"></knob>
</template>

<script>
import knob from './components/HelloWorld'
export default {
  name: 'app',
  data () {
    return {
      knobValue: 100
    }
  },
  components: {knob},
  methods: {
    knobChange (value) {
      console.log('--1', value)
    },
    send () {
      this.knobValue = 255
      console.log(this.knobValue)
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
