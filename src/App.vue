<template>
  <div>
    <Register
      v-if="currentView === 'register'"
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
import Register from './components/Register.vue'
import ChatBot from './components/ChatBot.vue'
import Admin from './components/Admin.vue'
import axios from 'axios'

export default {
  name: 'App',
  components: { Login, Register, ChatBot, Admin },
  data() {
    return {
      currentView: 'login',
      currentUser: null,
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
      this.currentView = 'login'
      this.sessions = []
      this.sidebarOpen = false
    },
    async loadHistory() {
      this.isLoadingHistory = true
      try {
        const token = localStorage.getItem('token')
        const response = await axios.get('http://127.0.0.1:8000/api/chat/history', {
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
  background: rgba(44, 24, 16, 0.4);
  z-index: 10;
  backdrop-filter: blur(3px);
}

.sidebar {
  position: fixed;
  top: 0;
  left: -280px;
  width: 270px;
  height: 100vh;
  background: #2c1810;
  display: flex;
  flex-direction: column;
  z-index: 20;
  transition: left 0.3s ease;
  color: #faf7f2;
  box-shadow: 4px 0 24px rgba(44, 24, 16, 0.4);
}
.sidebar.open {
  left: 0;
}

.sidebar-header {
  padding: 20px 16px;
  border-bottom: 1px solid rgba(201, 168, 76, 0.2);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.sidebar-logo {
  display: flex;
  align-items: center;
  gap: 10px;
}

.logo-icon {
  font-size: 24px;
  background: rgba(201, 168, 76, 0.15);
  border: 1px solid rgba(201, 168, 76, 0.3);
  border-radius: 10px;
  width: 42px;
  height: 42px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.sidebar-header h2 {
  font-size: 16px;
  font-weight: 700;
  letter-spacing: 2px;
  color: #c9a84c;
  font-family: 'Georgia', serif;
}

.sidebar-header p {
  font-size: 11px;
  color: rgba(250, 247, 242, 0.4);
  margin-top: 1px;
}

.close-btn {
  background: transparent;
  border: none;
  color: rgba(250, 247, 242, 0.4);
  font-size: 16px;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 6px;
}
.close-btn:hover {
  background: rgba(250, 247, 242, 0.08);
  color: #faf7f2;
}

.new-chat-btn {
  width: 100%;
  background: rgba(201, 168, 76, 0.12);
  border: 1px solid rgba(201, 168, 76, 0.3);
  color: #c9a84c;
  padding: 10px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 16px;
  text-align: left;
  transition: all 0.2s;
  font-family: 'Segoe UI', Arial, sans-serif;
}
.new-chat-btn:hover {
  background: rgba(201, 168, 76, 0.2);
}

.sidebar-history {
  flex: 1;
  overflow-y: auto;
  padding: 12px 8px;
}
.sidebar-history::-webkit-scrollbar {
  width: 4px;
}
.sidebar-history::-webkit-scrollbar-thumb {
  background: rgba(201, 168, 76, 0.2);
  border-radius: 4px;
}

.history-label {
  font-size: 10px;
  color: rgba(250, 247, 242, 0.3);
  text-transform: uppercase;
  padding: 0 8px;
  margin-bottom: 8px;
  letter-spacing: 1.5px;
  font-family: 'Segoe UI', Arial, sans-serif;
}

.history-loading,
.history-empty {
  font-size: 13px;
  color: rgba(250, 247, 242, 0.3);
  padding: 12px 8px;
  font-family: 'Segoe UI', Arial, sans-serif;
}

.history-item {
  padding: 10px 12px;
  border-radius: 8px;
  cursor: pointer;
  margin-bottom: 4px;
  transition: all 0.2s;
  border: 1px solid transparent;
}
.history-item:hover {
  background: rgba(201, 168, 76, 0.1);
  border-color: rgba(201, 168, 76, 0.2);
}
.history-item.active {
  background: rgba(201, 168, 76, 0.15);
  border-color: rgba(201, 168, 76, 0.3);
}

.history-message {
  font-size: 13px;
  color: rgba(250, 247, 242, 0.7);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-bottom: 3px;
  font-family: 'Segoe UI', Arial, sans-serif;
}

.history-meta {
  font-size: 11px;
  color: rgba(250, 247, 242, 0.3);
  font-family: 'Segoe UI', Arial, sans-serif;
}

.sidebar-footer {
  padding: 16px;
  border-top: 1px solid rgba(201, 168, 76, 0.15);
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.admin-btn {
  width: 100%;
  background: rgba(201, 168, 76, 0.12);
  border: 1px solid rgba(201, 168, 76, 0.3);
  color: #c9a84c;
  padding: 10px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 13px;
  font-weight: 600;
  text-align: left;
  transition: all 0.2s;
  font-family: 'Segoe UI', Arial, sans-serif;
}
.admin-btn:hover {
  background: rgba(201, 168, 76, 0.2);
}

.user-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 13px;
  color: rgba(250, 247, 242, 0.5);
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
  background: #c9a84c;
  border-radius: 50%;
}

.logout-btn {
  background: transparent;
  border: 1px solid rgba(250, 247, 242, 0.15);
  color: rgba(250, 247, 242, 0.5);
  padding: 4px 10px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.2s;
  font-family: 'Segoe UI', Arial, sans-serif;
}
.logout-btn:hover {
  background: rgba(250, 247, 242, 0.08);
  color: #faf7f2;
}

.main-area {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}
</style>
