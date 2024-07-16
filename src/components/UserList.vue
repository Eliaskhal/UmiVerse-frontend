<template>
  <div class="user-list">
    <h2>Connected Users</h2>
    <UserDisplay v-for="user in users" :key="user.userId" :user="user" />
  </div>
</template>

<script>
import UserDisplay from './UserDisplay.vue';

export default {
  components: {
    UserDisplay
  },
  props: {
    users: {
      type: Array,
      required: true
    }
  }
};
</script>

<style scoped>
.user-list {
  max-width: 600px;
  margin: 0 auto;
}

h2 {
  color: #007bff;
  font-size: 1.5em;
  margin-bottom: 1em;
}

/* No specific styles needed for UserList itself */
</style>
