<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'UmiVerse',
  mounted() {
    this.$router.push('/login');
  }
};
</script>

<style>
/* Your styles here */
</style>
