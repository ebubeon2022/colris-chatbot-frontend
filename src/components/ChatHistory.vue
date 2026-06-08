<template>
  <div class="history-panel">
    <h3>💬 Recent Conversations</h3>

    <div v-if="isLoading" class="history-loading">Loading history...</div>

    <div v-else-if="sessions.length === 0" class="history-empty">
      No previous conversations found
    </div>

    <div v-else class="history-list">
      <div
        v-for="session in sessions"
        :key="session.session_id"
        class="history-item"
        @click="$emit('load-session', session.session_id)"
      >
        <p class="history-message">{{ session.first_message }}</p>
        <span class="history-meta">
          {{ session.message_count }} messages · {{ formatDate(session.started_at) }}
        </span>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'ChatHistory',
  emits: ['load-session'],
  data() {
    return {
      sessions: [],
      isLoading: true,
    }
  },
  mounted() {
    this.loadHistory()
  },
  methods: {
    async loadHistory() {
      try {
        const token = localStorage.getItem('token')
        const response = await axios.get('http://127.0.0.1:8000/api/chat/history', {
          headers: {
            Authorization: `Bearer ${token}`,
            Accept: 'application/json',
          },
        })
        this.sessions = response.data.history
      } catch (error) {
        console.error('Failed to load history:', error)
      } finally {
        this.isLoading = false
      }
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString('en-GB', {
        day: 'numeric',
        month: 'short',
        hour: '2-digit',
        minute: '2-digit',
      })
    },
  },
}
</script>

<style scoped>
.history-panel {
  width: 260px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  padding: 20px;
  height: fit-content;
  max-height: 600px;
  overflow-y: auto;
}

.history-panel h3 {
  color: #1a3c5e;
  margin-bottom: 16px;
  font-size: 16px;
}

.history-loading {
  color: #999;
  font-size: 14px;
  text-align: center;
  padding: 20px 0;
}

.history-empty {
  color: #999;
  font-size: 14px;
  text-align: center;
  padding: 20px 0;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.history-item {
  padding: 12px;
  border: 1px solid #eee;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
}

.history-item:hover {
  background-color: #f0f4f8;
  border-color: #1a3c5e;
}

.history-message {
  font-size: 13px;
  color: #333;
  margin: 0 0 6px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.history-meta {
  font-size: 11px;
  color: #999;
}
</style>
