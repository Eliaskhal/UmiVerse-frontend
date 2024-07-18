<template>
  <div class="dashboard-container">
    <h2>Dashboard</h2>
    <p>Welcome, {{ userName }}</p>
    <button @click="disconnect">Disconnect</button>
    <p v-if="message" class="message">{{ message }}</p>
    <br>
    <h2>Online Users</h2>
    <ul>
      <li v-for="user in onlineUsers" :key="user.id">
        {{ user.id }} is {{ user.online ? 'online' : 'offline' }}
      </li>
    </ul>
  </div>
</template>

<script>
import SockJS from "sockjs-client";
import Stomp from "stompjs";

export default {
  data() {
    return {
      message: '',
      userName: '',
      stompClient: null,
      onlineUsers: []
    };
  },
  created(){
    this.connect();
  },
  mounted() {
    const userID = this.getCookie('userID');
    if (userID) {
      this.fetchUserDetails(userID);
    } else {
      this.$router.push('/');
    }
  },
  methods: {
    async fetchUserDetails(userID) {
      try {
        const response = await fetch(`http://localhost:8080/api/users/details?id=${userID}`);
        const data = await response.json();
        if (response.ok) {
          this.userName = data.username;
        } else {
          throw new Error(data.message || 'Failed to fetch user details');
        }
      } catch (error) {
        console.error('Error:', error);
        this.message = error.message || 'Failed to fetch user details';
      }
    },
    async disconnect() {
      try {
        const userID = this.getCookie('userID');
        const response = fetch(`http://localhost:8080/api/users/disconnect?id=${userID}`, {
          method: 'POST'
        });

        const data = response.json;

        console.log('User disconnected', data);
        this.body = 'User disconnected';

        document.cookie = `userID=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;`;
        this.$router.push('/');
      } catch (error) {
        console.error('Error:', error.message);
        this.message = error.message || 'Failed to disconnect';
      }
    },
    connect(){
      const socket = new SockJS("http://localhost:8080/ws")
      this.stompClient = Stomp.over(socket)

      this.stompClient.debug = (str) => {
        console.log(new Date(), str);
      }

      this.stompClient.connect({},
          this.onConnected, this.onError
      )
    },
    onConnected(){
      console.log('connected!!')
      this.stompClient.subscribe(
          '/topic/online',
          this.onMessageReceived
      )
    },
    onError(error){
      console.error('Could not connect to WebSocket server!', error);
    },
    onMessageReceived(payload){
      const message = JSON.parse(payload.body);
      this.updateUserStatus(message);
      console.log("Message received")
    },
    updateUserStatus(message){
      const existingUser = this.onlineUsers.find(
          user => user.id === message.id
      );
      if (existingUser) {
        existingUser.online = message.online;
      } else {
        this.onlineUsers.push(message);
        console.log(this.onlineUsers);
      }
    },
    getCookie(name) {
      const value = `; ${document.cookie}`;
      const parts = value.split(`; ${name}=`);
      if (parts.length === 2) return parts.pop().split(';').shift();
    }
  }
};
</script>

<style scoped>
.dashboard-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 1em;
}

button {
  padding: 0.75em;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

.message {
  color: red;
  margin-top: 1em;
}
</style>
