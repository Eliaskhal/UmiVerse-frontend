<!-- src/App.vue -->
<template>
  <div id="app" class="login">
    <Login />
  </div>
</template>

<script>
import Login from './components/Login.vue';

export default {
  components: {
    Login
  }
};
</script>

<style>
.login {
  left: 40%;
  width: 20%;
}
</style>
