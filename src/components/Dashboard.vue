<template>
  <div class="dashboard-container">
    <h2>Dashboard</h2>
    <UserList :users="connectedUsers" />
  </div>
</template>

<script>
import UserList from './UserList.vue';

export default {
  components: {
    UserList
  },
  data() {
    return {
      connectedUsers: []
    };
  },
  created() {
    this.fetchConnectedUsers();
  },
  methods: {
    async fetchConnectedUsers() {
      try {
        const response = await fetch('http://localhost:8088/api/users/online');
        if (!response.ok) {
          throw new Error('Failed to fetch connected users');
        }
        const data = await response.json();
        this.connectedUsers = data; // Assuming data is an array of users
      } catch (error) {
        console.error('Error:', error.message);
        // Handle error
      }
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

h2 {
  color: #007bff;
  font-size: 2em;
  margin-bottom: 1em;
}
</style>
