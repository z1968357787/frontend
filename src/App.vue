<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'app'
}
</script>

<style>
  *{
    margin:0;
    padding: 0;
  }
  body,html{
    height: 100%;
    overflow-y:auto;
  }
</style>
