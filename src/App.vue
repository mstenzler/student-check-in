<template>
  <div id="app">
    <StudentCheckIn user="Mike"/>
  </div>
</template>

<script>
import StudentCheckIn from './components/StudentCheckIn'

export default {
  name: 'App',
  components: {
    StudentCheckIn
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
</style>
