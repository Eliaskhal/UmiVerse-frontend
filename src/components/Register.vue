<template>
  <div class="register-container">
    <h2>Register</h2>
    <form @submit.prevent="handleSubmit">
      <div class="form-group">
        <label for="username">Username</label>
        <input type="text" id="username" v-model="username" required />
      </div>
      <div class="form-group">
        <label for="password">Password</label>
        <input type="password" id="password" v-model="password" required />
      </div>
      <div class="form-group">
        <label for="email">Email</label>
        <input type="email" id="email" v-model="email" required />
      </div>
      <div class="form-group">
        <label for="fullname">Full Name</label>
        <input type="text" id="fullname" v-model="fullname" required />
      </div>
      <div class="form-group">
        <label>Role</label><br>
        <input type="radio" id="student" v-model="role" value="student" required>
        <label for="student">Student</label><br>
        <input type="radio" id="professor" v-model="role" value="professor">
        <label for="professor">Professor</label><br>
      </div>
      <button type="submit">Register</button>
    </form>
    <router-link to="/login">Already have an account? Login here</router-link>
    <p v-if="message" class="message">{{ message }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: '',
      password: '',
      email: '',
      fullname: '',
      role: 'student', // Default role
      message: ''
    };
  },
  methods: {
    async handleSubmit() {
      try {
        const response = await fetch('http://localhost:8088/api/users/register', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            username: this.username,
            password: this.password,
            email: this.email,
            fullname: this.fullname,
            role: this.role
          })
        });

        const responseData = await response.json();

        if (!response.ok) {
          throw new Error(responseData.message || 'Registration failed');
        }

        console.log('Registration successful', responseData);
        this.message = 'Registration successful';

        // Save userID in a cookie
        document.cookie = `userID=${responseData.userID}`;

        // Redirect to dashboard
        this.$router.push('/dashboard');
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

.form-group {
  margin-bottom: 1em;
}

label {
  display: block;
  margin-bottom: 0.5em;
}

input {
  width: 100%;
  padding: 0.5em;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  width: 100%;
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

/* Styling for router-link */
.router-link {
  display: block;
  margin-top: 1em;
  text-decoration: none;
  color: #007bff;
}
</style>
