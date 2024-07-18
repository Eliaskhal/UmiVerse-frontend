<template>
  <div class="login-container">
    <h2>Login</h2>
    <form @submit.prevent="loginUser">
      <label for="username">Username:</label>
      <input type="text" id="username" v-model="username" required>
      <br><br>
      <label for="password">Password:</label>
      <input type="password" id="password" v-model="password" required>
      <br><br>
      <button type="submit">Login</button>
    </form>
    <p v-if="message" class="message">{{ message }}</p>
    <p>Don't have an account? <router-link to="/register">Register here</router-link></p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: '',
      password: '',
      message: ''
    };
  },
  mounted() {
    const userID = this.getCookie('userID');
    if (userID) {
      this.$router.push('/dashboard');
    }
  },
  methods: {
    async loginUser() {
      try {
        const response = await fetch('http://localhost:8080/api/users/login', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            username: this.username,
            password: this.password
          })
        });

        const data = await response.json();
        if (response.ok) {
          document.cookie = `userID=${data.code}`;
          this.$router.push('/dashboard');
        } else {
          throw new Error(data.message || 'Login failed');
        }
      } catch (error) {
        console.error('Error:', error);
        this.message = error.message || 'Login failed';
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
.login-container {
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
  color: red;
  margin-top: 1em;
}
</style>
