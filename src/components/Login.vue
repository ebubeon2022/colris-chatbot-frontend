<template>
  <div class="login-container">
    <div class="login-left">
      <div class="brand">
        <div class="logo-icon">📚</div>
        <h1>COLRIS</h1>
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
        <div class="card-header">
          <h2>Welcome Back</h2>
          <p class="subtitle">Sign in to your COLRIS account</p>
        </div>

        <div class="input-group">
          <label>Email Address</label>
          <input v-model="email" type="email" placeholder="your@stu.cu.edu.ng" @keyup.enter="login" />
        </div>

        <div class="input-group">
          <label>Password</label>
          <div class="password-wrapper">
            <input v-model="password" :type="showPassword ? 'text' : 'password'" placeholder="Enter your password" @keyup.enter="login" />
            <button type="button" @click="showPassword = !showPassword" class="eye-btn">
              <span v-if="showPassword"><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"/><line x1="1" y1="1" x2="23" y2="23"/></svg></span>
              <span v-else><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/></svg></span>
            </button>
          </div>
        </div>

        <div v-if="!showForgot && errorMessage" class="error-message">⚠️ {{ errorMessage }}</div>

        <div v-if="showForgot" class="forgot-box">
          <p class="forgot-title">Reset Password</p>
          <p class="forgot-sub">Enter your CU email and we'll send you a reset code.</p>
          <div class="input-group">
            <label>Email Address</label>
            <input v-model="resetEmail" type="email" placeholder="your@stu.cu.edu.ng" />
          </div>
          <div v-if="resetMessage" :class="['reset-msg', resetSuccess ? 'success' : 'error']">{{ resetMessage }}</div>
          <button @click="sendReset" class="login-btn" :disabled="isResetting">
            <span v-if="!isResetting">Send Reset Code →</span>
            <span v-else class="loading-dots"><span></span><span></span><span></span></span>
          </button>
          <p class="switch-link" style="margin-top:12px;"><span @click="showForgot = false">← Back to Sign In</span></p>
        </div>

        <button @click="login" class="login-btn" :disabled="isLoading">
          <span v-if="!isLoading">Sign In →</span>
          <span v-else class="loading-dots"><span></span><span></span><span></span></span>
        </button>

        <p v-if="!showForgot" class="switch-link">
          <span @click="showForgot = true" class="forgot-link">Forgot password?</span>
        </p>
        <p v-if="!showForgot" class="switch-link">
          Don't have an account? <span @click="$emit('show-register')">Register here</span>
        </p>
        <p class="switch-link" style="margin-top:8px;">
          <span @click="$emit('show-landing')" class="forgot-link">← Back to Home</span>
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
      showForgot: false,
      resetEmail: '',
      resetMessage: '',
      resetSuccess: false,
      isResetting: false,
    }
  },
  methods: {
    async sendReset() {
      this.resetMessage = ''
      this.isResetting = true
      try {
        await fetch('https://colris-chatbot-backend-production.up.railway.app/api/forgot-password', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json', 'Accept': 'application/json' },
          body: JSON.stringify({ email: this.resetEmail })
        })
        this.resetSuccess = true
        this.resetMessage = 'If that email exists, a reset code has been sent. Check your inbox.'
      } catch (e) {
        this.resetSuccess = false
        this.resetMessage = 'Something went wrong. Please try again.'
      } finally {
        this.isResetting = false
      }
    },
    async login() {
      this.errorMessage = ''
      this.isLoading = true
      try {
        const response = await axios.post(
          'https://colris-chatbot-backend-production.up.railway.app/api/login',
          { email: this.email, password: this.password },
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
* { box-sizing: border-box; }

.login-container {
  min-height: 100vh;
  display: flex;
  font-family: 'Segoe UI', Arial, sans-serif;
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

.login-left {
  flex: 1;
  background: linear-gradient(160deg, #2c1810 0%, #5c3d2e 60%, #8b5e3c 100%);
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 60px;
  color: white;
  position: relative;
  overflow: hidden;
}

.login-left::before {
  content: '';
  position: absolute;
  top: -120px; right: -120px;
  width: 420px; height: 420px;
  background: rgba(201, 168, 76, 0.08);
  border-radius: 50%;
  animation: pulse 4s ease-in-out infinite;
}

.login-left::after {
  content: '';
  position: absolute;
  bottom: -80px; left: -80px;
  width: 300px; height: 300px;
  background: rgba(201, 168, 76, 0.06);
  border-radius: 50%;
  animation: pulse 4s ease-in-out infinite reverse;
}

@keyframes pulse { 0%, 100% { transform: scale(1); opacity: 1; } 50% { transform: scale(1.08); opacity: 0.7; } }

.brand {
  margin-bottom: 48px;
  position: relative;
  z-index: 1;
  animation: slideUp 0.6s ease both;
}

@keyframes slideUp { from { opacity: 0; transform: translateY(24px); } to { opacity: 1; transform: translateY(0); } }

.logo-icon {
  font-size: 36px;
  width: 72px; height: 72px;
  background: rgba(201, 168, 76, 0.2);
  border-radius: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  border: 1px solid rgba(201, 168, 76, 0.35);
  transition: transform 0.3s, box-shadow 0.3s;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
}

.logo-icon:hover { transform: scale(1.08) rotate(-3deg); box-shadow: 0 8px 30px rgba(201, 168, 76, 0.25); }

.brand h1 {
  font-size: 38px;
  font-weight: 800;
  letter-spacing: 4px;
  margin: 0 0 8px;
  color: #fdf6e3;
}

.brand p { font-size: 14px; color: rgba(253, 246, 227, 0.65); margin: 0; }

.features {
  display: flex;
  flex-direction: column;
  gap: 12px;
  position: relative;
  z-index: 1;
}

.feature {
  font-size: 14px;
  color: rgba(253, 246, 227, 0.85);
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 18px;
  background: rgba(255, 255, 255, 0.07);
  border-radius: 12px;
  border: 1px solid rgba(201, 168, 76, 0.15);
  transition: all 0.25s ease;
  cursor: default;
  animation: slideUp 0.6s ease both;
}

.feature:nth-child(1) { animation-delay: 0.1s; }
.feature:nth-child(2) { animation-delay: 0.2s; }
.feature:nth-child(3) { animation-delay: 0.3s; }
.feature:nth-child(4) { animation-delay: 0.4s; }

.feature:hover {
  background: rgba(201, 168, 76, 0.12);
  border-color: rgba(201, 168, 76, 0.35);
  transform: translateX(8px);
  color: #fdf6e3;
}

.login-right {
  flex: 1;
  background: #faf7f2;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px;
}

.login-card {
  width: 100%;
  max-width: 400px;
  background: white;
  border-radius: 24px;
  padding: 40px;
  box-shadow: 0 8px 40px rgba(92, 61, 46, 0.1);
  border: 1px solid #f0e8dc;
  animation: slideUp 0.5s ease both;
  transition: box-shadow 0.3s;
}

.login-card:hover { box-shadow: 0 16px 56px rgba(92, 61, 46, 0.15); }

.card-header { margin-bottom: 32px; }

.login-card h2 {
  color: #2c1810;
  font-size: 26px;
  font-weight: 800;
  margin: 0 0 6px;
  font-family: 'Georgia', serif;
}

.subtitle { color: #8b7355; font-size: 14px; margin: 0; }

.input-group { margin-bottom: 20px; }

.input-group label {
  display: block;
  color: #5c3d2e;
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 8px;
  transition: color 0.2s;
}

.input-group:focus-within label { color: #c9a84c; }

.input-group input {
  width: 100%;
  padding: 13px 16px;
  background: #faf7f2;
  border: 1.5px solid #e8dcc8;
  border-radius: 10px;
  color: #2c1810;
  font-size: 14px;
  outline: none;
  transition: all 0.25s ease;
}

.input-group input::placeholder { color: #c4b49a; }

.input-group input:focus {
  border-color: #c9a84c;
  background: white;
  box-shadow: 0 0 0 3px rgba(201, 168, 76, 0.15);
  transform: translateY(-1px);
}

.forgot-link { color: #8b5e3c; cursor: pointer; font-weight: 600; font-size: 13px; }
.forgot-link:hover { text-decoration: underline; }
.forgot-box { background: #faf7f2; border: 1.5px solid #e8dcc8; border-radius: 12px; padding: 20px; margin-bottom: 16px; }
.forgot-title { color: #1a0f0a; font-size: 16px; font-weight: 700; margin: 0 0 4px; }
.forgot-sub { color: #8b7355; font-size: 13px; margin: 0 0 16px; }
.reset-msg { padding: 10px 14px; border-radius: 8px; font-size: 13px; font-weight: 600; margin-bottom: 12px; }
.reset-msg.success { background: #f0fdf4; border: 1px solid #bbf7d0; color: #16a34a; }
.reset-msg.error { background: #fef2f2; border: 1px solid #fecaca; color: #dc2626; }
.password-wrapper { position: relative; display: flex; align-items: center; }
.password-wrapper input { flex: 1; padding-right: 44px; }

.eye-btn {
  position: absolute; right: 12px;
  background: none; border: none;
  cursor: pointer; color: #8b7355;
  padding: 0; display: flex; align-items: center;
  transition: color 0.2s, transform 0.2s;
}
.eye-btn:hover { color: #5c3d2e; transform: scale(1.15); }

.error-message {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #dc2626;
  padding: 12px 16px;
  border-radius: 10px;
  font-size: 13px;
  margin-bottom: 16px;
  animation: shake 0.4s ease;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20% { transform: translateX(-6px); }
  40% { transform: translateX(6px); }
  60% { transform: translateX(-4px); }
  80% { transform: translateX(4px); }
}

.login-btn {
  width: 100%;
  padding: 14px;
  background: linear-gradient(135deg, #5c3d2e 0%, #8b5e3c 100%);
  color: #fdf6e3;
  border: none;
  border-radius: 12px;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.25s ease;
  box-shadow: 0 4px 16px rgba(92, 61, 46, 0.25);
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 50px;
  position: relative;
  overflow: hidden;
}

.login-btn::before {
  content: '';
  position: absolute;
  top: 0; left: -100%;
  width: 100%; height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.1), transparent);
  transition: left 0.4s ease;
}

.login-btn:hover::before { left: 100%; }

.login-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 28px rgba(92, 61, 46, 0.35);
  background: linear-gradient(135deg, #2c1810 0%, #5c3d2e 100%);
}

.login-btn:active { transform: translateY(0); box-shadow: 0 2px 8px rgba(92, 61, 46, 0.2); }

.login-btn:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }

.loading-dots { display: flex; gap: 5px; align-items: center; }
.loading-dots span {
  width: 7px; height: 7px;
  background: #fdf6e3;
  border-radius: 50%;
  animation: bounce 1.2s infinite ease-in-out;
}
.loading-dots span:nth-child(2) { animation-delay: 0.2s; }
.loading-dots span:nth-child(3) { animation-delay: 0.4s; }
@keyframes bounce { 0%, 60%, 100% { transform: translateY(0); } 30% { transform: translateY(-6px); } }

.switch-link {
  text-align: center;
  color: #8b7355;
  font-size: 13px;
  margin: 20px 0 0;
}

.switch-link span {
  color: #8b5e3c;
  cursor: pointer;
  font-weight: 700;
  transition: color 0.2s;
  text-decoration: underline;
  text-underline-offset: 3px;
}

.switch-link span:hover { color: #5c3d2e; }

@media (max-width: 768px) {
  .login-left { display: none; }
  .login-right { width: 100%; }
  .login-card { padding: 28px 20px; }
}
</style>
