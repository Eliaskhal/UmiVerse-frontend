<template>
  <div class="user-display">
    <p><strong>Username:</strong> {{ user.username }}</p>
    <p><strong>Full Name:</strong> {{ user.fullName }}</p>
  </div>
</template>

<script>
export default {
  props: {
    user: {
      type: Object,
      required: true
    }
  }
};
</script>

<style scoped>
.user-display {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 1em;
  margin-bottom: 1em;
  background-color: #f0f0f0;
}
</style>
