<template>
  <div class="dashboard-container">
    <h2>Dashboard</h2>
    <p v-if="userName">Welcome, {{ userName }}</p>
    <div v-if="connectedUsers.length > 0">
      <h3>Connected Users</h3>
      <ul>
        <li v-for="user in connectedUsers" :key="user.userID">
          {{ user.username }} ({{ user.fullName }})
        </li>
      </ul>
    </div>
    <button @click="disconnect">Disconnect</button>
    <p v-if="message" class="message">{{ message }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: '',
      userName: '',
      connectedUsers: []
    };
  },
  mounted() {
    // Check if userID cookie exists, fetch user details if it does
    const userID = this.getCookie('userID');
    if (userID) {
      this.fetchUserDetails(userID);
      this.fetchConnectedUsers(userID);
    } else {
      // Redirect to login if userID cookie is not found
      this.$router.push('/login');
    }
  },
  methods: {
    async fetchUserDetails(userID) {
      try {
        const response = await fetch(`http://localhost:8088/api/users/details?id=${userID}`);
        const data = await response.json();

        if (response.status !== 302) {
          throw new Error(data.message || 'Failed to fetch user details');
        }

        // Assuming the response contains a 'username' field
        this.userName = data.username;
      } catch (error) {
        console.error('Error:', error);
        this.message = error.message || 'Failed to fetch user details';
      }
    },
    async fetchConnectedUsers(userID) {
      try {
        const response = await fetch(`http://localhost:8088/api/users/online`);
        const data = await response.json();

        if (!response.ok) {
          throw new Error(data.message || 'Failed to fetch connected users');
        }

        // Filter out the current user from connected users
        this.connectedUsers = data.filter(user => user.userID !== parseInt(userID));
      } catch (error) {
        console.error('Error:', error);
        this.message = error.message || 'Failed to fetch connected users';
      }
    },
    async disconnect() {
      try {
        const userID = this.getCookie('userID');
        const response = await fetch(`http://localhost:8088/api/users/disconnect?id=${userID}`, {
          method: 'POST'
        });

        if (!response.ok) {
          throw new Error('Failed to disconnect');
        }

        // Clear user details and cookie
        this.userName = '';
        document.cookie = `userID=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;`;

        // Redirect to login page
        this.$router.push('/login');
      } catch (error) {
        console.error('Error:', error.message);
        this.message = error.message || 'Failed to disconnect';
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
  max-width: 400px;
  margin: 0 auto;
  padding: 1em;
  border: 1px solid #ccc;
  border-radius: 4px;
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
  color: green;
  margin-top: 1em;
}
</style>
