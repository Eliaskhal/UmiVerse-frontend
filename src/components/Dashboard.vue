<template>
  <div class="dashboard-container">
    <h2>Dashboard</h2>
    <p>Welcome, {{ userName }}</p>
    <button @click="disconnect">Disconnect</button>
    <p v-if="message" class="message">{{ message }}</p>
    <div>
      <h3>Active Users</h3>
      <ul>
        <li v-for="user in activeUsers" :key="user.userId">
          {{ user.username }}
          <button @click="sendMessage(user.userId)">Send Message</button>
        </li>
      </ul>
    </div>
    <div>
      <h3>Received Messages</h3>
      <ul>
        <li v-for="receivedMessage in receivedMessages" :key="receivedMessage.id">
          <strong>{{ receivedMessage.sender }}</strong>: {{ receivedMessage.content }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import SockJS from 'sockjs-client';
import Stomp from 'stompjs';

export default {
  data() {
    return {
      message: '',
      userName: '',
      activeUsers: [],
      receivedMessages: [],
      stompClient: null
    };
  },
  mounted() {
    const userID = this.getCookie('userID');
    if (userID) {
      this.fetchUserDetails(userID);
      this.initWebsocket();
    } else {
      this.$router.push('/login');
    }
  },
  methods: {
    async fetchUserDetails(userID) {
      try {
        const response = await fetch(`http://localhost:8088/api/users/details?id=${userID}`);
        const data = await response.json();
        if (response.ok) {
          this.userName = data.username;
          await this.fetchActiveUsers();
        } else {
          throw new Error(data.message || 'Failed to fetch user details');
        }
      } catch (error) {
        console.error('Error:', error);
        this.message = error.message || 'Failed to fetch user details';
      }
    },
    async fetchActiveUsers() {
      try {
        const response = await fetch(`http://localhost:8088/api/users/online`);
        const data = await response.json();
        if (response.ok) {
          this.activeUsers = data.filter(user => user.userId !== this.getCookie('userID'));
        } else {
          throw new Error(data.message || 'Failed to fetch active users');
        }
      } catch (error) {
        console.error('Error:', error);
        this.message = error.message || 'Failed to fetch active users';
      }
    },
    disconnect() {
      try {
        const userID = this.getCookie('userID');
        const response = fetch(`http://localhost:8088/api/users/disconnect?id=${userID}`, {
          method: 'POST'
        });

        if (!response.ok) {
          throw new Error('Failed to disconnect');
        }

        const data = response.json;

        console.log('User disconnected', data);
        this.body = 'User disconnected';

        document.cookie = `userID=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;`;
        this.$router.push('/login');
      } catch (error) {
        console.error('Error:', error.message);
        this.message = error.message || 'Failed to disconnect';
      }
    },
    initWebsocket() {
      const socket = new SockJS('http://localhost:8088/ws');
      this.stompClient = Stomp.over(socket);
      this.stompClient.connect({}, () => {
        console.log('Connected to websocket');
        this.subscribeToMessages();
        this.fetchActiveUsers();
      }, (error) => {
        console.error('Websocket error:', error);
      });
    },
    subscribeToMessages() {
      this.stompClient.subscribe(`/user/${this.getCookie('userID')}/queue/messages`, (message) => {
        console.log('Received message:', message.body);
        this.receivedMessages.push(JSON.parse(message.body));
      });
    },
    sendMessage(recipientId) {
      const messageContent = 'Hello, let’s chat!';
      this.stompClient.send(`/app/chat`, {}, JSON.stringify({
        senderId: this.getCookie('userID'),
        recipientId: recipientId,
        content: messageContent
      }));
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
