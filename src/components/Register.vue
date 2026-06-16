<template>
  <div class="register-container">
    <div class="register-left">
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

    <div class="register-right">
      <div class="register-card">

        <div v-if="step === 'register'">
          <div class="card-header">
            <h2>Create Account</h2>
            <p class="subtitle">Join the COLRIS Library system</p>
          </div>

          <div class="input-group">
            <label>Full Name</label>
            <input v-model="name" type="text" placeholder="Enter your full name" @keyup.enter="register" />
          </div>

          <div class="input-group">
            <label>Email Address</label>
            <input v-model="email" type="email" placeholder="your@stu.cu.edu.ng" @keyup.enter="register" />
          </div>

          <div class="input-group">
            <label>Password</label>
            <div class="password-wrapper">
              <input v-model="password" :type="showPassword ? 'text' : 'password'" placeholder="Create a password (min. 6 characters)" @keyup.enter="register" />
              <button type="button" @click="showPassword = !showPassword" class="eye-btn">
                <span v-if="showPassword"><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"/><line x1="1" y1="1" x2="23" y2="23"/></svg></span>
                <span v-else><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/></svg></span>
              </button>
            </div>
            <div class="pass-hint" v-if="password.length > 0">
              <span :class="password.length >= 6 ? 'pass-ok' : 'pass-warn'">
                {{ password.length >= 6 ? '✓ Good length' : (6 - password.length) + ' more characters needed' }}
              </span>
            </div>
          </div>

          <div class="input-group">
            <label>Confirm Password</label>
            <div class="password-wrapper">
              <input v-model="passwordConfirmation" :type="showConfirm ? 'text' : 'password'" placeholder="Confirm your password" @keyup.enter="register" />
              <button type="button" @click="showConfirm = !showConfirm" class="eye-btn">
                <span v-if="showConfirm"><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"/><line x1="1" y1="1" x2="23" y2="23"/></svg></span>
                <span v-else><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/></svg></span>
              </button>
            </div>
          </div>

          <div class="divider"><span>Library Staff Only</span></div>

          <div class="input-group">
            <label>Admin Code <span class="optional">(staff only — leave blank if student)</span></label>
            <input v-model="adminCode" type="text" placeholder="Enter staff admin code" @keyup.enter="register" />
          </div>

          <div v-if="errorMessage" class="error-message">⚠️ {{ errorMessage }}</div>

          <button @click="register" class="register-btn" :disabled="isLoading">
            <span v-if="!isLoading">Create Account →</span>
            <span v-else class="loading-dots"><span></span><span></span><span></span></span>
          </button>

          <p class="switch-link">
            Already have an account? <span @click="$emit('show-login')">Sign In</span>
          </p>
          <p class="switch-link" style="margin-top:8px;">
            <span @click="$emit('show-landing')" style="color:#6d28d9;cursor:pointer;font-weight:600;">← Back to Home</span>
          </p>
        </div>

        <div v-else-if="step === 'otp'">
          <div class="card-header">
            <h2>Verify Your Email</h2>
            <p class="subtitle">We sent a 6-digit code to <strong>{{ email }}</strong></p>
          </div>
          <div class="input-group">
            <label>Enter OTP Code</label>
            <input v-model="otpCode" type="text" maxlength="6" placeholder="e.g. 123456" @keyup.enter="verifyOtp" style="letter-spacing:6px;font-size:20px;text-align:center;" />
          </div>
          <div v-if="errorMessage" class="error-message">⚠️ {{ errorMessage }}</div>
          <button @click="verifyOtp" class="register-btn" :disabled="isLoading">
            <span v-if="!isLoading">Verify Email →</span>
            <span v-else class="loading-dots"><span></span><span></span><span></span></span>
          </button>
          <p class="switch-link" style="margin-top:16px;">
            Didn't receive it?
            <span v-if="resendCooldown > 0" style="color:#b8a898;cursor:default;">Resend in {{ resendCooldown }}s</span>
            <span v-else @click="resendOtp">Resend OTP</span>
          </p>
          <p class="switch-link"><span @click="step = 'register'">← Back to Register</span></p>
        </div>

        <div v-else-if="step === 'success'" class="success-screen">
          <div class="success-icon">🎉</div>
          <h2>{{ registeredAsAdmin ? 'Admin Account Created!' : 'Welcome to COLRIS!' }}</h2>
          <p v-if="registeredAsAdmin">Your staff account has been created successfully. Please sign in with your credentials to access the admin panel.</p>
          <p v-else>Thank you for registering, <strong>{{ name }}</strong>! Your account is ready.</p>
          <div class="countdown">Redirecting to login in {{ countdown }}s...</div>
          <button @click="goToLogin" class="register-btn">Sign In Now →</button>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'Register',
  emits: ['login-success', 'show-login', 'show-landing'],
  data() {
    return {
      step: 'register',
      name: '',
      email: '',
      password: '',
      passwordConfirmation: '',
      adminCode: '',
      showPassword: false,
      showConfirm: false,
      errorMessage: '',
      isLoading: false,
      registeredAsAdmin: false,
      countdown: 5,
      countdownTimer: null,
      resendCooldown: 0,
      resendTimer: null,
    }
  },
  beforeUnmount() {
    if (this.countdownTimer) clearInterval(this.countdownTimer)
  },
  methods: {
    async register() {
      this.errorMessage = ''
      if (!this.name || !this.email || !this.password || !this.passwordConfirmation) {
        this.errorMessage = 'Please fill in all required fields.'
        return
      }
      if (this.password !== this.passwordConfirmation) {
        this.errorMessage = 'Passwords do not match!'
        return
      }
      if (this.password.length < 6) {
        this.errorMessage = 'Password must be at least 6 characters.'
        return
      }
      this.isLoading = true
      try {
        const payload = {
          name: this.name,
          email: this.email,
          password: this.password,
          password_confirmation: this.passwordConfirmation,
        }
        if (this.adminCode) payload.admin_code = this.adminCode
        await axios.post(
          'https://colris-chatbot-backend-production.up.railway.app/api/register',
          payload,
          { headers: { Accept: 'application/json' } },
        )
        this.step = 'otp'
        this.startResendCooldown()
      } catch (error) {
        this.errorMessage = error.response && error.response.data && error.response.data.message
          ? error.response.data.message
          : 'Registration failed. Please try again.'
      } finally {
        this.isLoading = false
      }
    },
    async verifyOtp() {
      this.errorMessage = ''
      if (!this.otpCode || this.otpCode.length !== 6) {
        this.errorMessage = 'Please enter the 6-digit code.'
        return
      }
      this.isLoading = true
      try {
        const response = await axios.post(
          'https://colris-chatbot-backend-production.up.railway.app/api/verify-otp',
          { email: this.email, otp: this.otpCode },
          { headers: { Accept: 'application/json' } }
        )
        this.registeredAsAdmin = response.data.user.role === 'admin'
        this.step = 'success'
        this.countdown = 5
        this.countdownTimer = setInterval(() => {
          this.countdown--
          if (this.countdown <= 0) this.goToLogin()
        }, 1000)
      } catch (error) {
        this.errorMessage = error.response && error.response.data ? error.response.data.message : 'Invalid OTP.'
      } finally {
        this.isLoading = false
      }
    },
    startResendCooldown() {
      if (this.resendTimer) clearInterval(this.resendTimer)
      this.resendCooldown = 60
      this.resendTimer = setInterval(() => {
        this.resendCooldown--
        if (this.resendCooldown <= 0) {
          clearInterval(this.resendTimer)
          this.resendTimer = null
          this.resendCooldown = 0
        }
      }, 1000)
    },
    async resendOtp() {
      try {
        await axios.post('https://colris-chatbot-backend-production.up.railway.app/api/resend-otp', { email: this.email }, { headers: { Accept: 'application/json' } })
        this.startResendCooldown()
      } catch (e) { console.error(e) }
    },
    goToLogin() {
      if (this.countdownTimer) clearInterval(this.countdownTimer)
      this.$emit('show-login')
    },
  },
}
</script>

<style scoped>
* { box-sizing: border-box; }

.register-container {
  min-height: 100vh;
  display: flex;
  font-family: 'Segoe UI', Arial, sans-serif;
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

.register-left {
  flex: 1;
  background: linear-gradient(160deg, #2d1457 0%, #5c3d2e 60%, #6d28d9 100%);
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 60px;
  color: white;
  position: relative;
  overflow: hidden;
}

.register-left::before {
  content: '';
  position: absolute;
  top: -120px; right: -120px;
  width: 420px; height: 420px;
  background: rgba(124, 58, 237, 0.08);
  border-radius: 50%;
  animation: pulse 4s ease-in-out infinite;
}

.register-left::after {
  content: '';
  position: absolute;
  bottom: -80px; left: -80px;
  width: 300px; height: 300px;
  background: rgba(124, 58, 237, 0.06);
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
  background: rgba(124, 58, 237, 0.2);
  border-radius: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  border: 1px solid rgba(124, 58, 237, 0.35);
  transition: transform 0.3s, box-shadow 0.3s;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
}

.logo-icon:hover { transform: scale(1.08) rotate(-3deg); box-shadow: 0 8px 30px rgba(124, 58, 237, 0.25); }

.brand h1 { font-size: 38px; font-weight: 800; letter-spacing: 4px; margin: 0 0 8px; color: #f5f3ff; }
.brand p { font-size: 14px; color: rgba(253, 246, 227, 0.65); margin: 0; }

.features { display: flex; flex-direction: column; gap: 12px; position: relative; z-index: 1; }

.feature {
  font-size: 14px;
  color: rgba(253, 246, 227, 0.85);
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 18px;
  background: rgba(255, 255, 255, 0.07);
  border-radius: 12px;
  border: 1px solid rgba(124, 58, 237, 0.15);
  transition: all 0.25s ease;
  cursor: default;
  animation: slideUp 0.6s ease both;
}

.feature:nth-child(1) { animation-delay: 0.1s; }
.feature:nth-child(2) { animation-delay: 0.2s; }
.feature:nth-child(3) { animation-delay: 0.3s; }
.feature:nth-child(4) { animation-delay: 0.4s; }

.feature:hover {
  background: rgba(124, 58, 237, 0.12);
  border-color: rgba(124, 58, 237, 0.35);
  transform: translateX(8px);
}

.register-right {
  flex: 1;
  background: #faf7f2;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px;
  overflow-y: auto;
}

.register-card {
  width: 100%;
  max-width: 420px;
  background: white;
  border-radius: 24px;
  padding: 40px;
  box-shadow: 0 8px 40px rgba(92, 61, 46, 0.1);
  border: 1px solid #f0e8dc;
  animation: slideUp 0.5s ease both;
  transition: box-shadow 0.3s;
}

.register-card:hover { box-shadow: 0 16px 56px rgba(92, 61, 46, 0.15); }

.card-header { margin-bottom: 28px; }

.register-card h2 { color: #2d1457; font-size: 26px; font-weight: 800; margin: 0 0 6px; font-family: 'Georgia', serif; }
.subtitle { color: #7c3aed; font-size: 14px; margin: 0; }

.input-group { margin-bottom: 18px; }

.input-group label {
  display: block;
  color: #5c3d2e;
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 7px;
  transition: color 0.2s;
}

.input-group:focus-within label { color: #7c3aed; }

.input-group input {
  width: 100%;
  padding: 12px 16px;
  background: #faf7f2;
  border: 1.5px solid #ddd6fe;
  border-radius: 10px;
  color: #2d1457;
  font-size: 14px;
  outline: none;
  transition: all 0.25s ease;
}

.input-group input::placeholder { color: #c4b49a; }

.input-group input:focus {
  border-color: #7c3aed;
  background: white;
  box-shadow: 0 0 0 3px rgba(124, 58, 237, 0.15);
  transform: translateY(-1px);
}

.password-wrapper { position: relative; display: flex; align-items: center; }
.password-wrapper input { flex: 1; padding-right: 44px; }

.eye-btn {
  position: absolute; right: 12px;
  background: none; border: none;
  cursor: pointer; color: #7c3aed;
  padding: 0; display: flex; align-items: center;
  transition: color 0.2s, transform 0.2s;
}
.eye-btn:hover { color: #5c3d2e; transform: scale(1.15); }

.pass-hint { margin-top: 6px; font-size: 12px; }
.pass-ok { color: #16a34a; font-weight: 600; }
.pass-warn { color: #dc2626; font-weight: 600; }

.divider {
  text-align: center;
  margin: 20px 0 16px;
  position: relative;
}
.divider::before {
  content: '';
  position: absolute;
  top: 50%; left: 0;
  width: 100%; height: 1px;
  background: #ddd6fe;
}
.divider span {
  background: white;
  padding: 0 12px;
  color: #7c3aed;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 1px;
  text-transform: uppercase;
  position: relative;
}

.optional { color: #b8a898; font-weight: 400; font-size: 11px; }

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

.register-btn {
  width: 100%;
  padding: 14px;
  background: linear-gradient(135deg, #5c3d2e 0%, #6d28d9 100%);
  color: #f5f3ff;
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

.register-btn::before {
  content: '';
  position: absolute;
  top: 0; left: -100%;
  width: 100%; height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.1), transparent);
  transition: left 0.4s ease;
}

.register-btn:hover::before { left: 100%; }

.register-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 28px rgba(92, 61, 46, 0.35);
  background: linear-gradient(135deg, #2d1457 0%, #5c3d2e 100%);
}

.register-btn:active { transform: translateY(0); }
.register-btn:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }

.loading-dots { display: flex; gap: 5px; align-items: center; }
.loading-dots span {
  width: 7px; height: 7px;
  background: #f5f3ff;
  border-radius: 50%;
  animation: bounce 1.2s infinite ease-in-out;
}
.loading-dots span:nth-child(2) { animation-delay: 0.2s; }
.loading-dots span:nth-child(3) { animation-delay: 0.4s; }
@keyframes bounce { 0%, 60%, 100% { transform: translateY(0); } 30% { transform: translateY(-6px); } }

.switch-link { text-align: center; color: #7c3aed; font-size: 13px; margin: 20px 0 0; }
.switch-link span {
  color: #6d28d9;
  cursor: pointer;
  font-weight: 700;
  text-decoration: underline;
  text-underline-offset: 3px;
  transition: color 0.2s;
}
.switch-link span:hover { color: #5c3d2e; }

.success-screen {
  text-align: center;
  padding: 20px 0;
  animation: slideUp 0.5s ease;
}

.success-icon {
  font-size: 56px;
  margin-bottom: 20px;
  animation: popIn 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

@keyframes popIn { from { transform: scale(0); opacity: 0; } to { transform: scale(1); opacity: 1; } }

.success-screen h2 { color: #2d1457; font-size: 22px; font-weight: 800; margin: 0 0 12px; font-family: 'Georgia', serif; }
.success-screen p { color: #7c3aed; font-size: 14px; line-height: 1.6; margin: 0 0 20px; }

.countdown {
  background: #f5f3ff;
  border: 1px solid #ddd6fe;
  color: #6d28d9;
  padding: 10px 16px;
  border-radius: 8px;
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 20px;
}

@media (max-width: 768px) {
  .register-left { display: none; }
  .register-right { width: 100%; }
  .register-card { padding: 28px 20px; }
}
</style>
