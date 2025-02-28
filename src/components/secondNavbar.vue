<template>
  <nav class="bottom-navbar">
    <button @click="openDialog" class="auth-button">Login / Register</button>

    <div class="brand">
      <img src="../assets/main logo.jpeg" alt="Main Logo" class="logo" />
    </div>

    <div class="icons">
      <button @click="toggleLanguage">
        <i class="fas fa-language"></i>
      </button>
      <button @click="toggleTheme">
        <i :class="isDark ? 'fas fa-sun' : 'fas fa-moon'"></i>
      </button>
    </div>

    <div v-if="isDialogOpen" class="dialog-backdrop" @click="closeDialog">
      <div class="dialog" @click.stop>
        <div class="form-toggle">
          <button @click="activeForm = 'login'" :class="{ active: activeForm === 'login' }">Login</button>
          <button @click="activeForm = 'register'" :class="{ active: activeForm === 'register' }">Register</button>
        </div>

        <form v-if="activeForm === 'login'" @submit.prevent="handleLogin">
          <input type="email" placeholder="Email" required />
          <input type="password" placeholder="Password" required />
          <button type="submit">Login</button>
        </form>

        <form v-if="activeForm === 'register'" @submit.prevent="handleSignup">
          <input type="text" placeholder="Full Name" required />
          <input type="email" placeholder="Email" required />
          <input type="password" placeholder="Password" required />
          <button type="submit">Register</button>
        </form>

        <button @click="closeDialog" class="close-button">Close</button>
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  name: "BottomNavbar",
  props: {
    isDark: Boolean, // Receive isDark from App.vue
  },
  data() {
    return {
      isDialogOpen: false,
      activeForm: "login",
      isEnglish: true,
    };
  },
  methods: {
    openDialog() {
      this.isDialogOpen = true;
    },
    closeDialog() {
      this.isDialogOpen = false;
    },
    toggleLanguage() {
      this.isEnglish = !this.isEnglish;
    },
    toggleTheme() {
      this.$emit("toggle-theme"); // Emit event to App.vue to change theme
    },
    handleLogin() {
      alert("Login successful");
    },
    handleSignup() {
      alert("Registration successful");
    },
  },
};
</script>

<style scoped>
.bottom-navbar {
  display: flex;
  justify-content: space-around;
  align-items: center;
  background-color: green;
  padding: 0.75rem 1rem;
  width: 100%;
  z-index: 1000;
}

/* Dark Mode */
.dark-mode .bottom-navbar {
  background-color: #222;
}

/* Logo Styling */
.brand {
  display: flex;
  align-items: center;
  justify-content: center;
}

.logo {
  width: 120px;
  height: auto;
  max-height: 50px;
  object-fit: contain;
}

.auth-button {
  padding: 0.5rem 1rem;
  border: none;
  background-color: #fff;
  color: green;
  border-radius: 4px;
  cursor: pointer;
}

.icons {
  display: flex;
  gap: 1rem;
}

.icons button {
  background: none;
  border: none;
  color: #fff;
  font-size: 1.25rem;
  cursor: pointer;
}

/* Dialog Styles */
.dialog-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
}

.dialog {
  background-color: #fff;
  width: 90%;
  max-width: 400px;
  padding: 1.5rem;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.form-toggle {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.form-toggle button {
  padding: 0.5rem 1rem;
  border: none;
  background: none;
  color: #333;
  cursor: pointer;
  font-weight: bold;
}

.form-toggle button.active {
  border-bottom: 2px solid green;
  color: green;
}

.dialog form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.dialog input {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.dialog button[type="submit"] {
  padding: 0.5rem;
  border: none;
  background-color: green;
  color: #fff;
  border-radius: 4px;
  cursor: pointer;
}

.close-button {
  margin-top: 1rem;
  width: 100%;
  padding: 0.5rem;
  border: none;
  background-color: #ccc;
  color: #333;
  border-radius: 4px;
  cursor: pointer;
}

/* Dark Mode Styles */
.dark-mode {
  background-color: #333;
  color: #fff;
}

.dark-mode .dialog {
  background-color: #444;
  color: #fff;
}

.dark-mode .dialog input {
  background-color: #555;
  color: #fff;
  border-color: #666;
}

.car-types-container{
  width: 80%;
}
</style>
