<template>
  <div class="login-container">
    <div class="login-left">
      <div class="brand">
        <div class="logo-icon">🤖</div>
        <h1>COLRIS AI</h1>
        <p>Covenant University Library Assistant</p>
      </div>
      <div class="features">
        <div class="feature">📚 Search millions of books</div>
        <div class="feature">🤖 AI-powered library help</div>
        <div class="feature">🔗 Real-time book links</div>
        <div class="feature">💬 24/7 library assistance</div>
      </div>
    </div>

    <div class="login-right">
      <div class="login-card">
        <h2>Welcome Back</h2>
        <p class="subtitle">Sign in to your COLRIS account</p>

        <div class="input-group">
          <label>Email Address</label>
          <input
            v-model="email"
            type="email"
            placeholder="your@stu.cu.edu.ng"
            @keyup.enter="login"
          />
            <button type="button" @click="showPassword = !showPassword" class="eye-btn">{{ showPassword ? '🙈' : '👁️' }}</button>
            </div>
        </div>

        <div class="input-group">
          <label>Password</label>
          <input
            v-model="password"
            :type="showPassword ? 'text' : 'password'"
            placeholder="Enter your password"
            @keyup.enter="login"
          />
            <button type="button" @click="showPassword = !showPassword" class="eye-btn">{{ showPassword ? '🙈' : '👁️' }}</button>
            </div>
        </div>

        <div v-if="errorMessage" class="error-message">⚠️ {{ errorMessage }}</div>

        <button @click="login" class="login-btn" :disabled="isLoading">
          {{ isLoading ? 'Signing in...' : 'Sign In →' }}
        </button>

        <p class="switch-link">
          Don't have an account? <span @click="$emit('show-register')">Register here</span>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Login',
  emits: ['login-success', 'show-register'],
  data() {
    return {
      email: '',
      password: '',
      showPassword: false,
      errorMessage: '',
      isLoading: false,
    }
  },
  methods: {
    async login() {
      this.errorMessage = ''
      this.isLoading = true
      try {
        const response = await axios.post(
          'https://colris-chatbot-backend-production.up.railway.app/api/login',
          {
            email: this.email,
            password: this.password,
          },
          { headers: { Accept: 'application/json' } },
        )
        localStorage.setItem('token', response.data.token)
        localStorage.setItem('user', JSON.stringify(response.data.user))
        this.$emit('login-success', response.data)
      } catch (error) {
        this.errorMessage = 'Invalid email or password. Please try again.'
      } finally {
        this.isLoading = false
      }
    },
  },
}
</script>

<style scoped>
.login-container {
  min-height: 100vh;
  display: flex;
  font-family: 'Segoe UI', Arial, sans-serif;
}

.login-left {
  flex: 1;
  background: linear-gradient(160deg, #0d1b3e 0%, #1e3a6e 60%, #1e6fd9 100%);
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 60px;
  color: white;
}

.brand {
  margin-bottom: 48px;
}

.logo-icon {
  font-size: 48px;
  width: 80px;
  height: 80px;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.brand h1 {
  font-size: 36px;
  font-weight: 800;
  letter-spacing: 3px;
  margin: 0 0 8px;
  color: #f5f0e8;
}

.brand p {
  font-size: 15px;
  color: rgba(245, 240, 232, 0.6);
  margin: 0;
}

.features {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.feature {
  font-size: 15px;
  color: rgba(245, 240, 232, 0.8);
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 18px;
  background: rgba(255, 255, 255, 0.07);
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.login-right {
  width: 480px;
  background: #f5f0e8;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px;
}

.login-card {
  width: 100%;
  max-width: 380px;
}

.login-card h2 {
  color: #0d1b3e;
  font-size: 28px;
  font-weight: 800;
  margin: 0 0 6px;
}

.subtitle {
  color: #6b7280;
  font-size: 14px;
  margin: 0 0 32px;
}

.input-group {
  margin-bottom: 20px;
}

.input-group label {
  display: block;
  color: #0d1b3e;
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 8px;
}

.input-group input {
  width: 100%;
  padding: 14px 18px;
  background: white;
  border: 1.5px solid #e2e8f0;
  border-radius: 12px;
  color: #0d1b3e;
  font-size: 14px;
  outline: none;
  transition: all 0.2s;
  box-sizing: border-box;
}

.input-group input::placeholder {
  color: #adb5bd;
}

.input-group input:focus {
  border-color: #1e6fd9;
  box-shadow: 0 0 0 3px rgba(30, 111, 217, 0.12);
}

.error-message {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #dc2626;
  padding: 12px 16px;
  border-radius: 10px;
  font-size: 13px;
  margin-bottom: 16px;
}

.login-btn {
  width: 100%;
  padding: 15px;
  background: linear-gradient(135deg, #0d1b3e, #1e6fd9);
  color: #f5f0e8;
  border: none;
  border-radius: 12px;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s;
  box-shadow: 0 4px 16px rgba(13, 27, 62, 0.2);
}

.login-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(13, 27, 62, 0.3);
}

.login-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.switch-link {
  text-align: center;
  color: #6b7280;
  font-size: 13px;
  margin: 20px 0 0;
}

.switch-link span {
  color: #1e6fd9;
  cursor: pointer;
  font-weight: 600;
}

.switch-link span:hover {
  text-decoration: underline;
}

@media (max-width: 768px) {
  .login-left {
    display: none;
  }
  .login-right {
    width: 100%;
  }
}

.password-wrapper { position: relative; display: flex; align-items: center; }
.password-wrapper input { flex: 1; padding-right: 44px; }
.eye-btn { position: absolute; right: 12px; background: none; border: none; cursor: pointer; font-size: 16px; padding: 0; line-height: 1; }
</style>
