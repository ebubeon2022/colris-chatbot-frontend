<template>
  <div>
    <Landing
      v-if="currentView === 'landing'"
      @show-login="currentView = 'login'"
      @show-register="currentView = 'register'"
    />
    <Register
      v-else-if="currentView === 'register'"
      @login-success="handleLoginSuccess"
      @show-login="currentView = 'login'"
    />
    <Login
      v-else-if="currentView === 'login'"
      @login-success="handleLoginSuccess"
      @show-register="currentView = 'register'"
    />
    <Admin v-else-if="currentView === 'admin'" @back="currentView = 'chat'" />
    <div v-else class="app-layout">
      <div v-if="sidebarOpen" class="sidebar-overlay" @click="sidebarOpen = false"></div>

      <div class="sidebar" :class="{ open: sidebarOpen }">
        <div class="sidebar-header">
          <div class="sidebar-logo">
            <div class="logo-icon">📚</div>
            <div>
              <h2>COLRIS</h2>
              <p>Library Assistant</p>
            </div>
          </div>
          <button @click="sidebarOpen = false" class="close-btn">✕</button>
        </div>

        <div class="sidebar-history">
          <button @click="startNewChat" class="new-chat-btn">+ New Conversation</button>
          <p class="history-label">Recent Conversations</p>
          <div v-if="isLoadingHistory" class="history-loading">Loading...</div>
          <div v-else-if="sessions.length === 0" class="history-empty">No conversations yet</div>
          <div
            v-else
            v-for="session in sessions"
            :key="session.session_id"
            class="history-item"
            :class="{ active: selectedSession === session.session_id }"
            @click="loadSession(session.session_id)"
          >
            <p class="history-message">{{ session.first_message }}</p>
            <span class="history-meta">{{ formatDate(session.started_at) }}</span>
          </div>
        </div>

        <div class="sidebar-footer">
          <button v-if="currentUser && currentUser.role === 'admin'" @click="goToAdmin" class="admin-btn">
            Admin Panel
          </button>
          <div class="user-info">
            <div class="user-avatar-name">
              <div class="user-dot"></div>
              <span>{{ currentUser && currentUser.name ? currentUser.name : '' }}</span>
            </div>
            <button @click="handleLogout" class="logout-btn">Sign Out</button>
          </div>
        </div>
      </div>

      <div class="main-area">
        <ChatBot
          :user="currentUser"
          :key="chatKey"
          :session-id="selectedSession"
          @open-sidebar="sidebarOpen = true"
          @new-session="handleNewSession"
        />
      </div>
    </div>
  </div>
</template>

<script>
import Login from './components/Login.vue'
import Landing from './components/Landing.vue'
import Register from './components/Register.vue'
import ChatBot from './components/ChatBot.vue'
import Admin from './components/Admin.vue'
import axios from 'axios'

export default {
  name: 'App',
  components: { Login, Register, ChatBot, Admin },
  data() {
    return {
      currentView: 'landing',
      currentUser: null,
      showLanding: true,
      selectedSession: null,
      chatKey: 0,
      sessions: [],
      isLoadingHistory: false,
      sidebarOpen: false,
    }
  },
  mounted() {
    const token = localStorage.getItem('token')
    const user = localStorage.getItem('user')
    if (token && user) {
      this.currentUser = JSON.parse(user)
      this.currentView = 'chat'
      this.loadHistory()
    }
  },
  methods: {
    handleLoginSuccess(data) {
      this.currentUser = data.user
      this.currentView = 'chat'
      this.loadHistory()
    },
    handleLogout() {
      localStorage.removeItem('token')
      localStorage.removeItem('user')
      this.currentUser = null
      this.currentView = 'landing'
      this.sessions = []
      this.sidebarOpen = false
    },
    async loadHistory() {
      this.isLoadingHistory = true
      try {
        const token = localStorage.getItem('token')
        const response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/chat/history', {
          headers: { Authorization: `Bearer ${token}`, Accept: 'application/json' },
        })
        this.sessions = response.data.history
      } catch (error) {
        console.error('Failed to load history:', error)
      } finally {
        this.isLoadingHistory = false
      }
    },
    loadSession(sessionId) {
      this.selectedSession = sessionId
      this.sidebarOpen = false
    },
    startNewChat() {
      this.chatKey++
      this.selectedSession = null
      this.sidebarOpen = false
    },
    handleNewSession(sessionId) {
      this.selectedSession = sessionId
      this.loadHistory()
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString('en-GB', {
        day: 'numeric',
        month: 'short',
        hour: '2-digit',
        minute: '2-digit',
      })
    },
    goToAdmin() {
      this.currentView = 'admin'
      this.sidebarOpen = false
    },
  },
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  background: #faf7f2;
  min-height: 100vh;
  font-family: 'Georgia', serif;
}

.app-layout {
  display: flex;
  height: 100vh;
  overflow: hidden;
  position: relative;
}

.sidebar-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
  z-index: 10;
  backdrop-filter: blur(4px);
}

.sidebar {
  position: fixed;
  top: 0;
  left: -300px;
  width: 280px;
  height: 100vh;
  background: #1e1410;
  display: flex;
  flex-direction: column;
  z-index: 20;
  transition: left 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  color: #faf7f2;
  box-shadow: 8px 0 40px rgba(0, 0, 0, 0.5);
}
.sidebar.open {
  left: 0;
}

.sidebar-header {
  padding: 20px 18px;
  border-bottom: 1px solid rgba(255,255,255,0.06);
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #2c1810;
}

.sidebar-logo {
  display: flex;
  align-items: center;
  gap: 12px;
}

.logo-icon {
  font-size: 22px;
  background: rgba(201, 168, 76, 0.15);
  border: 1px solid rgba(201, 168, 76, 0.25);
  border-radius: 10px;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.3s;
}
.logo-icon:hover { transform: rotate(-5deg) scale(1.08); }

.sidebar-header h2 {
  font-size: 15px;
  font-weight: 800;
  letter-spacing: 3px;
  color: #ffffff;
  font-family: 'Georgia', serif;
}

.sidebar-header p {
  font-size: 10px;
  color: rgba(255,255,255,0.35);
  margin-top: 2px;
  letter-spacing: 0.3px;
}

.close-btn {
  background: transparent;
  border: none;
  color: rgba(255,255,255,0.5);
  font-size: 18px;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 6px;
  transition: all 0.2s;
  line-height: 1;
}
.close-btn:hover {
  background: rgba(255,255,255,0.07);
  color: white;
}

.new-chat-btn {
  width: 100%;
  background: #c9a84c;
  border: none;
  color: #1a0f0a;
  padding: 11px 14px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 800;
  margin-bottom: 20px;
  text-align: center;
  transition: all 0.25s;
  font-family: 'Segoe UI', Arial, sans-serif;
  letter-spacing: 0.3px;
  box-shadow: 0 2px 12px rgba(201,168,76,0.3);
}
.new-chat-btn:hover {
  background: #b8960a;
  transform: translateY(-1px);
  box-shadow: 0 4px 20px rgba(201,168,76,0.4);
}

.sidebar-history {
  flex: 1;
  overflow-y: auto;
  padding: 16px 12px;
}
.sidebar-history::-webkit-scrollbar {
  width: 3px;
}
.sidebar-history::-webkit-scrollbar-thumb {
  background: rgba(255,255,255,0.08);
  border-radius: 4px;
}

.history-label {
  font-size: 10px;
  color: rgba(255,255,255,0.45);
  text-transform: uppercase;
  padding: 0 8px;
  margin-bottom: 10px;
  letter-spacing: 2px;
  font-family: 'Segoe UI', Arial, sans-serif;
}

.history-loading,
.history-empty {
  font-size: 13px;
  color: rgba(255,255,255,0.45);
  padding: 16px 8px;
  font-family: 'Segoe UI', Arial, sans-serif;
  text-align: center;
}

.history-item {
  padding: 10px 12px;
  border-radius: 8px;
  cursor: pointer;
  margin-bottom: 3px;
  transition: all 0.2s;
  border: 1px solid transparent;
}
.history-item:hover {
  background: rgba(255,255,255,0.05);
  border-color: rgba(255,255,255,0.08);
}
.history-item.active {
  background: rgba(201,168,76,0.12);
  border-color: rgba(201,168,76,0.25);
}

.history-message {
  font-size: 13px;
  color: rgba(255,255,255,0.82);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-bottom: 3px;
  font-family: 'Segoe UI', Arial, sans-serif;
}

.history-meta {
  font-size: 11px;
  color: rgba(255,255,255,0.45);
  font-family: 'Segoe UI', Arial, sans-serif;
}

.sidebar-footer {
  padding: 16px;
  border-top: 1px solid rgba(255,255,255,0.06);
  display: flex;
  flex-direction: column;
  gap: 10px;
  background: #2c1810;
}

.admin-btn {
  width: 100%;
  background: transparent;
  border: 1px solid rgba(201,168,76,0.25);
  color: #c9a84c;
  padding: 10px 14px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 700;
  text-align: left;
  transition: all 0.2s;
  font-family: 'Segoe UI', Arial, sans-serif;
  letter-spacing: 0.3px;
}
.admin-btn:hover {
  background: #c9a84c;
  color: #1a0f0a;
  border-color: #c9a84c;
}

.user-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 13px;
  color: rgba(255,255,255,0.6);
  font-family: 'Segoe UI', Arial, sans-serif;
}

.user-avatar-name {
  display: flex;
  align-items: center;
  gap: 8px;
}

.user-dot {
  width: 8px;
  height: 8px;
  background: #22c55e;
  border-radius: 50%;
  box-shadow: 0 0 6px rgba(34,197,94,0.5);
}

.logout-btn {
  background: transparent;
  border: 1px solid rgba(255,255,255,0.1);
  color: rgba(255,255,255,0.6);
  padding: 5px 12px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.2s;
  font-family: 'Segoe UI', Arial, sans-serif;
}
.logout-btn:hover {
  background: rgba(220,38,38,0.15);
  border-color: rgba(220,38,38,0.3);
  color: #ef4444;
}

.main-area {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}
</style>

