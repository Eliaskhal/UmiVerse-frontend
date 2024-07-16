<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import SockJS from 'sockjs-client';
import Stomp from 'stompjs';
export default {
  name: 'App'
};
</script>

<style>
/* Your styles here */
</style>
