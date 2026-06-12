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
        <!-- REGISTRATION FORM -->
        <div v-if="step === 'register'">
          <h2>Create Account</h2>
          <p class="subtitle">Join the COLRIS Library system</p>

          <div class="input-group">
            <label>Full Name</label>
            <input
              v-model="name"
              type="text"
              placeholder="Enter your full name"
              @keyup.enter="register"
            />
          </div>

          <div class="input-group">
            <label>Email Address</label>
            <input
              v-model="email"
              type="email"
              placeholder="your@stu.cu.edu.ng"
              @keyup.enter="register"
            />
          </div>

          <div class="input-group">
            <label>Password</label>
            <div class="password-wrapper">
              <input
                v-model="password"
                :type="showPassword ? 'text' : 'password'"
                placeholder="Create a password (min. 6 characters)"
                @keyup.enter="register"
              />
              <button type="button" @click="showPassword = !showPassword" class="eye-btn"><span v-if="showPassword"><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"/><line x1="1" y1="1" x2="23" y2="23"/></svg></span><span v-else><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/></svg></span></button>
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
              <input
                v-model="passwordConfirmation"
                :type="showConfirm ? 'text' : 'password'"
                placeholder="Confirm your password"
                @keyup.enter="register"
              />
              <button type="button" @click="showConfirm = !showConfirm" class="eye-btn"><span v-if="showConfirm"><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"/><line x1="1" y1="1" x2="23" y2="23"/></svg></span><span v-else><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/></svg></span></button>
            </div>
          </div>

          <div class="divider"><span>Library Staff Only</span></div>

          <div class="input-group">
            <label
              >Admin Code <span class="optional">(staff only — leave blank if student)</span></label
            >
            <input
              v-model="adminCode"
              type="text"
              placeholder="Enter staff admin code"
              @keyup.enter="register"
            />
          </div>

          <div v-if="errorMessage" class="error-message">⚠️ {{ errorMessage }}</div>

          <button @click="register" class="register-btn" :disabled="isLoading">
            {{ isLoading ? 'Creating Account...' : 'Create Account →' }}
          </button>

          <p class="switch-link">
            Already have an account? <span @click="$emit('show-login')">Sign in</span>
          </p>
        </div>

        <!-- OTP VERIFICATION SCREEN -->
        <div v-else-if="step === 'otp'" class="otp-screen">
          <div class="otp-icon">📧</div>
          <h2>Check Your Email</h2>
          <p class="subtitle">We sent a 6-digit verification code to</p>
          <p class="otp-email">{{ email }}</p>

          <div class="input-group">
            <label>Verification Code</label>
            <input
              v-model="otpCode"
              type="text"
              placeholder="Enter 6-digit code"
              maxlength="6"
              class="otp-input"
              @keyup.enter="verifyOtp"
            />
          </div>

          <div v-if="errorMessage" class="error-message">⚠️ {{ errorMessage }}</div>

          <button @click="verifyOtp" class="register-btn" :disabled="isLoading">
            {{ isLoading ? 'Verifying...' : 'Verify Email →' }}
          </button>

          <div class="resend-row">
            <span>Didn't receive the code?</span>
            <button @click="resendOtp" class="resend-btn" :disabled="resendCooldown > 0">
              {{ resendCooldown > 0 ? `Resend in ${resendCooldown}s` : 'Resend Code' }}
            </button>
          </div>

          <p class="switch-link">
            <span @click="step = 'register'">← Back to Register</span>
          </p>
        </div>

        <!-- SUCCESS SCREEN -->
        <div v-else-if="step === 'success'" class="success-screen">
          <div class="success-icon">🎉</div>
          <h2>{{ registeredAsAdmin ? 'Admin Account Created!' : 'Welcome to COLRIS!' }}</h2>
          <p v-if="registeredAsAdmin">
            Your staff account has been created successfully. Please sign in with your credentials
            to access the admin panel.
          </p>
          <p v-else>
            Thank you for registering, <strong>{{ name }}</strong
            >! Your account is verified and ready.
          </p>
          <div class="countdown">Redirecting to login in {{ countdown }}s...</div>
          <button @click="goToLogin" class="login-now-btn">Sign In Now →</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Register',
  emits: ['login-success', 'show-login'],
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
      otpCode: '',
      errorMessage: '',
      isLoading: false,
      registeredAsAdmin: false,
      countdown: 5,
      countdownTimer: null,
      resendCooldown: 0,
      resendTimer: null,
    }
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

      this.isLoading = true
      try {
        const payload = {
          name: this.name,
          email: this.email,
          password: this.password,
          password_confirmation: this.passwordConfirmation,
        }
        if (this.adminCode) {
          payload.admin_code = this.adminCode
        }

        const regResponse = await axios.post('https://colris-chatbot-backend-production.up.railway.app/api/register', payload, {
          headers: { Accept: 'application/json' },
        })

        this.registeredAsAdmin = regResponse.data.user.role === 'admin'
        this.step = 'success'; this.countdown = 5; this.countdownTimer = setInterval(() => { this.countdown--; if (this.countdown <= 0) { this.goToLogin() } }, 1000)
      } catch (error) {
        this.errorMessage =
          error.response?.data?.message || 'Registration failed. Please try again.'
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
          {
            email: this.email,
            otp: this.otpCode,
          },
          { headers: { Accept: 'application/json' } },
        )

        this.registeredAsAdmin = response.data.user.role === 'admin'
        this.step = 'success'; this.countdown = 5; this.countdownTimer = setInterval(() => { this.countdown--; if (this.countdown <= 0) { this.goToLogin() } }, 1000)

        this.countdown = 5
        this.countdownTimer = setInterval(() => {
          this.countdown--
          if (this.countdown <= 0) {
            this.goToLogin()
          }
        }, 1000)
      } catch (error) {
        this.errorMessage = error.response?.data?.message || 'Invalid OTP. Please try again.'
      } finally {
        this.isLoading = false
      }
    },

    async resendOtp() {
      try {
        await axios.post(
          'https://colris-chatbot-backend-production.up.railway.app/api/resend-otp',
          {
            email: this.email,
          },
          { headers: { Accept: 'application/json' } },
        )
        this.errorMessage = ''
      } catch (error) {
        this.errorMessage = 'Failed to resend OTP. Please try again.'
      }
    },

    startResendCooldown() {
      this.resendCooldown = 60
      this.resendTimer = setInterval(() => {
        this.resendCooldown--
        if (this.resendCooldown <= 0) {
          clearInterval(this.resendTimer)
        }
      }, 1000)
    },

    goToLogin() {
      if (this.countdownTimer) clearInterval(this.countdownTimer)
      this.$emit('show-login')
    },
  },

  beforeUnmount() {
    if (this.countdownTimer) clearInterval(this.countdownTimer)
    if (this.resendTimer) clearInterval(this.resendTimer)
  },
}
</script>

<style scoped>
.register-container {
  min-height: 100vh;
  display: flex;
  font-family: 'Segoe UI', Arial, sans-serif;
}

.register-left {
  flex: 1;
  background: linear-gradient(160deg, #2c1810 0%, #5c3d2e 60%, #8b5e3c 100%);
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
  background: rgba(201, 168, 76, 0.15);
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  border: 1px solid rgba(201, 168, 76, 0.3);
}
.brand h1 {
  font-size: 36px;
  font-weight: 800;
  letter-spacing: 3px;
  margin: 0 0 8px;
  color: #c9a84c;
  font-family: 'Georgia', serif;
}
.brand p {
  font-size: 15px;
  color: rgba(250, 247, 242, 0.6);
  margin: 0;
}

.features {
  display: flex;
  flex-direction: column;
  gap: 16px;
}
.feature {
  font-size: 15px;
  color: rgba(250, 247, 242, 0.8);
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 18px;
  background: rgba(201, 168, 76, 0.08);
  border-radius: 12px;
  border: 1px solid rgba(201, 168, 76, 0.15);
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
  max-width: 380px;
}

.register-card h2 {
  color: #2c1810;
  font-size: 28px;
  font-weight: 800;
  margin: 0 0 6px;
  font-family: 'Georgia', serif;
}
.subtitle {
  color: #8b7355;
  font-size: 14px;
  margin: 0 0 28px;
}

.input-group {
  margin-bottom: 16px;
}
.input-group label {
  display: block;
  color: #5c3d2e;
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 8px;
}
.optional {
  color: #b8a898;
  font-weight: 400;
  font-size: 11px;
}
.input-group input {
  width: 100%;
  padding: 14px 18px;
  background: white;
  border: 1.5px solid #e8dcc8;
  border-radius: 10px;
  color: #2c1810;
  font-size: 14px;
  outline: none;
  transition: all 0.2s;
  box-sizing: border-box;
}
.input-group input::placeholder {
  color: #b8a898;
}
.input-group input:focus {
  border-color: #c9a84c;
  box-shadow: 0 0 0 3px rgba(201, 168, 76, 0.15);
}

.divider {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 8px 0 16px;
}
.divider::before,
.divider::after {
  content: '';
  flex: 1;
  height: 1px;
  background: #e8dcc8;
}
.divider span {
  color: #b8a898;
  font-size: 12px;
  white-space: nowrap;
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

.register-btn {
  width: 100%;
  padding: 15px;
  background: #5c3d2e;
  color: #faf7f2;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s;
}
.register-btn:hover {
  background: #2c1810;
  transform: translateY(-2px);
}
.register-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.switch-link {
  text-align: center;
  color: #8b7355;
  font-size: 13px;
  margin: 20px 0 0;
}
.switch-link span {
  color: #8b5e3c;
  cursor: pointer;
  font-weight: 600;
}
.switch-link span:hover {
  text-decoration: underline;
}

/* OTP Screen */
.otp-screen {
  text-align: center;
}
.otp-icon {
  font-size: 56px;
  margin-bottom: 16px;
}
.otp-screen h2 {
  color: #2c1810;
  font-size: 26px;
  font-weight: 800;
  margin: 0 0 8px;
  font-family: 'Georgia', serif;
}
.otp-email {
  color: #5c3d2e;
  font-weight: 700;
  font-size: 14px;
  margin: 0 0 24px;
  background: #fdf6e3;
  padding: 8px 16px;
  border-radius: 8px;
  border: 1px solid #e8dcc8;
  display: inline-block;
}
.otp-input {
  text-align: center;
  font-size: 24px;
  font-weight: 700;
  letter-spacing: 8px;
  color: #2c1810;
  font-family: monospace;
}
.resend-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  margin: 16px 0;
  font-size: 13px;
  color: #8b7355;
}
.resend-btn {
  background: transparent;
  border: none;
  color: #8b5e3c;
  font-weight: 600;
  cursor: pointer;
  font-size: 13px;
  text-decoration: underline;
}
.resend-btn:disabled {
  color: #b8a898;
  cursor: not-allowed;
  text-decoration: none;
}

/* Success Screen */
.success-screen {
  text-align: center;
  padding: 20px 0;
}
.success-icon {
  font-size: 64px;
  margin-bottom: 20px;
}
.success-screen h2 {
  color: #2c1810;
  font-size: 24px;
  font-weight: 800;
  margin: 0 0 12px;
  font-family: 'Georgia', serif;
}
.success-screen p {
  color: #8b7355;
  font-size: 14px;
  line-height: 1.6;
  margin: 0 0 24px;
}
.success-screen strong {
  color: #2c1810;
}
.countdown {
  background: #fdf6e3;
  border: 1px solid #e8dcc8;
  color: #8b5e3c;
  padding: 10px 16px;
  border-radius: 10px;
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 16px;
}
.login-now-btn {
  width: 100%;
  padding: 15px;
  background: #5c3d2e;
  color: #faf7f2;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s;
}
.login-now-btn:hover {
  background: #2c1810;
  transform: translateY(-2px);
}

@media (max-width: 768px) {
  .register-left {
    display: none;
  }
  .register-right {
    width: 100%;
  }
}

.password-wrapper { position: relative; display: flex; align-items: center; }
.password-wrapper input { flex: 1; padding-right: 44px; }
.eye-btn { position: absolute; right: 12px; background: none; border: none; cursor: pointer; font-size: 12px; font-weight: 600; color: #8b7355; padding: 0; line-height: 1; letter-spacing: 0.5px; }
.pass-hint { margin-top: 5px; font-size: 12px; }
.pass-ok { color: #16a34a; font-weight: 600; }
.pass-warn { color: #dc2626; font-weight: 600; }
</style>
