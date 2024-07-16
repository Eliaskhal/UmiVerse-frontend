<template>
  <div class="dashboard-container">
    <h2>Dashboard</h2>
    <button @click="disconnect">Disconnect</button>
    <p v-if="message" class="message">{{ message }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: ''
    };
  },
  methods: {
    async disconnect() {
      const userId = localStorage.getItem('userId'); // Retrieve userId from local storage

      if (!userId) {
        this.message = 'User ID not found';
        return;
      }

      try {
        const response = await fetch(`http://localhost:8088/api/users/disconnect?id=${userId}`, {
          method: 'POST'
        });

        if (!response.ok) {
          throw new Error('Failed to disconnect');
        }

        const data = await response.text();
        console.log('User disconnected', data);
        this.message = data;
        localStorage.removeItem('userId'); // Optionally remove userId from local storage
        this.$router.push('/'); // Redirect to login
      } catch (error) {
        console.error('Error:', error.message);
        this.message = error.message;
      }
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
