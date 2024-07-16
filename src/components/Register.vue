<template>
  <div class="register-container">
    <h2>Register</h2>
    <form @submit.prevent="registerUser">
      <label for="username">Username:</label>
      <input type="text" id="username" v-model="username" required>
      <br><br>
      <label for="email">Email:</label>
      <input type="email" id="email" v-model="email" required>
      <br><br>
      <label for="fullName">Full Name:</label>
      <input type="text" id="fullName" v-model="fullName" required>
      <br><br>
      <label>Role:</label>
      <input type="radio" id="student" v-model="role" value="student" required>
      <label for="student">Student</label>
      <input type="radio" id="professor" v-model="role" value="professor">
      <label for="professor">Professor</label>
      <br><br>
      <label for="password">Password:</label>
      <input type="password" id="password" v-model="password" required>
      <br><br>
      <button type="submit">Register</button>
    </form>
    <p v-if="message" class="message">{{ message }}</p>
    <p>Already have an account? <router-link to="/login">Login here</router-link></p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: '',
      email: '',
      fullName: '',
      role: 'student',
      password: '',
      message: ''
    };
  },
  methods: {
    async registerUser() {
      try {
        const response = await fetch('http://localhost:8088/api/users/register', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            username: this.username,
            email: this.email,
            fullName: this.fullName,
            role: this.role,
            password: this.password
          })
        });

        const data = await response.json();
        if (response.ok) {
          document.cookie = `userID=${data.code}`;
          this.$router.push('/dashboard');
        } else {
          throw new Error(data.message || 'Registration failed');
        }
      } catch (error) {
        console.error('Error:', error);
        this.message = error.message || 'Registration failed';
      }
    }
  }
};
</script>

<style scoped>
.register-container {
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
