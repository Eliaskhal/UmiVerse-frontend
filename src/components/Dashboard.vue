<template>
  <div class="dashboard-container">
    <h2>Dashboard</h2>
    <p>Welcome, {{ userName }}</p>
    <button @click="disconnect">Disconnect</button>
    <p v-if="message" class="message">{{ message }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: '',
      userName: ''
    };
  },
  mounted() {
    const userID = this.getCookie('userID');
    if (userID) {
      this.fetchUserDetails(userID);
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
        } else {
          throw new Error(data.message || 'Failed to fetch user details');
        }
      } catch (error) {
        console.error('Error:', error);
        this.message = error.message || 'Failed to fetch user details';
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
