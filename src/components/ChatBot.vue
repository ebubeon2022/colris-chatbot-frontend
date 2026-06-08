<template>
  <div class="chat-container">
    <div class="chat-header">
      <button @click="$emit('open-sidebar')" class="menu-btn">☰</button>
      <div class="header-content">
        <div class="header-avatar">📚</div>
        <div>
          <h2>COLRIS Library Assistant</h2>
          <p class="header-status">
            <span class="status-dot"></span> Online • Welcome, {{ user && user.name ? user.name : 'Student' }}
          </p>
        </div>
      </div>
      <button @click="showSearch = !showSearch; showArrivals = false" class="search-toggle-btn">
        {{ showSearch ? 'Chat' : 'Search Books' }}
      </button>
    </div>

    <div v-if="announcement" class="announcement-banner">
      <span class="announcement-dot"></span>
      <span class="announcement-label">Notice</span>
      <span class="announcement-divider">|</span>
      <span>{{ announcement }}</span>
    </div>

    <div v-if="showSearch" class="search-panel">
      <div class="search-bar">
        <input v-model="searchQuery" @keyup.enter="searchBooks" placeholder="Search by title, author or subject..." type="text" />
        <button @click="searchBooks" class="search-btn" :disabled="isSearching">{{ isSearching ? '...' : 'Search' }}</button>
      </div>
      <div v-if="isSearching" class="search-loading">
        <div class="loading-dots"><span></span><span></span><span></span></div>
        <p>Searching library catalogue...</p>
      </div>
      <div v-else-if="searchResults.length === 0 && hasSearched" class="no-results">No books found for "{{ searchQuery }}"</div>
      <div v-else class="book-grid">
        <div v-for="book in searchResults" :key="book.link" class="book-card">
          <div class="book-cover">
            <img v-if="book.cover" :src="book.cover" :alt="book.title" />
            <div v-else class="no-cover">📚</div>
          </div>
          <div class="book-info">
            <p class="source-badge">{{ book.source }}</p>
            <h4 class="book-title">{{ book.title }}</h4>
            <p class="book-author">{{ book.author }}</p>
            <p class="book-year">{{ book.year }} · {{ book.publisher }}</p>
            <div class="book-subjects" v-if="book.subjects.length">
              <span v-for="subject in book.subjects" :key="subject" class="subject-tag">{{ subject }}</span>
            </div>
            <div class="book-actions">
              <a :href="book.link" target="_blank" class="view-btn">View</a>
              <a v-if="book.downloadable && book.download_link" :href="book.download_link" target="_blank" class="download-btn">Download PDF</a>
              <span class="access-badge" :class="'access-' + book.access_color">{{ book.access_label }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="showArrivals" class="search-panel">
      <div class="arrivals-header">
        <div>
          <h3 class="arrivals-title">Latest Arrivals</h3>
          <p class="arrivals-sub">New books recently added to the Covenant University Library</p>
        </div>
        <div class="arrivals-actions">
          <a href="https://clr.covenantuniversity.edu.ng" target="_blank" class="clr-link-btn">Visit CLR Website</a>
          <button @click="showArrivals = false" class="close-arrivals-btn">Back to Chat</button>
        </div>
      </div>
      <div v-if="isLoadingArrivals" class="search-loading">
        <div class="loading-dots"><span></span><span></span><span></span></div>
        <p>Loading latest arrivals...</p>
      </div>
      <div v-else-if="newArrivals.length === 0" class="no-results">No new arrivals tagged yet. Ask your librarian to update the catalogue!</div>
      <div v-else class="arrivals-grid">
        <div v-for="book in newArrivals" :key="book.id" class="arrival-card">
          <div class="arrival-badge">NEW</div>
          <div class="arrival-icon">📗</div>
          <div class="arrival-info">
            <h4 class="arrival-title">{{ book.title }}</h4>
            <p class="arrival-author">{{ book.author }}</p>
            <p class="arrival-category" v-if="book.category">{{ book.category }}</p>
            <div class="arrival-meta">
              <span class="arrival-location" v-if="book.location">📍 {{ book.location }}</span>
              <span class="arrival-stock" :class="book.stock > 0 ? 'in-stock' : 'out-stock'">
                {{ book.stock > 0 ? book.stock + ' copies available' : 'Out of stock' }}
              </span>
            </div>
            <a :href="colrisUrl(book.title)" target="_blank" class="arrival-colris-btn">Search in COLRIS</a>
          </div>
        </div>
      </div>
    </div>

    <div v-else class="chat-messages" ref="messageContainer">
      <div class="welcome-banner" v-if="messages.length === 1">
        <div class="welcome-icon">📚</div>
        <h3>How can I assist you today?</h3>
        <p class="welcome-sub">Ask me anything about the Covenant University Library</p>
        <div class="quick-actions">
          <button @click="quickAsk('What are the library opening hours?')">Library Hours</button>
          <button @click="quickAsk('How do I borrow a book?')">Borrow a Book</button>
          <button @click="quickAsk('What databases are available?')">Databases</button>
          <button @click="quickAsk('How do I access e-journals?')">E-Journals</button>
          <button @click="openLatestArrivals" class="arrivals-quick-btn">Latest Arrivals</button>
        </div>
      </div>
      <div v-for="(message, index) in messages" :key="index" :class="['message', message.sender]">
        <div class="message-avatar" v-if="message.sender === 'bot'">📚</div>
        <div class="message-bubble" v-html="formatMessage(message.text)"></div>
        <div class="message-avatar user-avatar" v-if="message.sender === 'user'">👤</div>
      </div>
      <div v-if="isLoading" class="message bot">
        <div class="message-avatar">📚</div>
        <div class="message-bubble typing-indicator"><span></span><span></span><span></span></div>
      </div>
    </div>

    <div v-if="!showSearch && !showArrivals" class="chat-input">
      <button @click="toggleVoice" class="mic-btn" :class="{ recording: isRecording }">
        <span v-if="!isRecording">🎤</span>
        <span v-else>⏹</span>
      </button>
      <input v-model="userInput" @keyup.enter="sendMessage" placeholder="Type your question here..." type="text" :disabled="isLoading" />
      <button @click="openLatestArrivals" class="arrivals-btn" title="Latest Arrivals">🆕</button>
      <button @click="sendMessage" :disabled="isLoading" class="send-btn">
        <span v-if="!isLoading">➤</span>
        <span v-else>...</span>
      </button>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'ChatBot',
  emits: ['logout', 'open-sidebar', 'new-session'],
  props: {
    user: Object,
    sessionId: String,
  },
  data() {
    return {
      userInput: '',
      isLoading: false,
      isRecording: false,
      recognition: null,
      currentSessionId: null,
      showSearch: false,
      showArrivals: false,
      searchQuery: '',
      searchResults: [],
      isSearching: false,
      hasSearched: false,
      announcement: '',
      newArrivals: [],
      isLoadingArrivals: false,
      messages: [
        { sender: 'bot', text: 'Hello! I am the COLRIS Library AI Assistant. How can I help you today?' },
      ],
    }
  },
  watch: {
    sessionId(newSessionId) {
      if (newSessionId) {
        this.loadSessionMessages(newSessionId)
      } else {
        this.resetChat()
      }
    },
  },
  async mounted() {
    try {
      const response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/settings/public')
      if (response.data.library_announcement) {
        this.announcement = response.data.library_announcement
      }
    } catch (e) {
      console.error(e)
    }
  },
  methods: {
    colrisUrl(title) {
      var encoded = encodeURIComponent(title)
      return 'https://colris.covenantuniversity.edu.ng/discovery/search?query=any,contains,' + encoded + '&tab=Everything&search_scope=MyInst_and_CI&vid=234COU_INST:VU1&lang=en&offset=0'
    },
    async openLatestArrivals() {
      this.showArrivals = true
      this.showSearch = false
      if (this.newArrivals.length === 0) {
        await this.loadNewArrivals()
      }
    },
    async loadNewArrivals() {
      this.isLoadingArrivals = true
      try {
        const response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/books/new-arrivals')
        this.newArrivals = response.data.books || []
      } catch (e) {
        console.error(e)
      } finally {
        this.isLoadingArrivals = false
      }
    },
    formatMessage(text) {
      if (!text) return ''
      var f = text.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')
      f = f.replace(/(https?:\/\/[^\s]+)/g, '<a href="$1" target="_blank" rel="noopener noreferrer">$1</a>')
      f = f.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
      f = f.replace(/^(\d+)\.\s(.+)$/gm, '<div class="list-item"><span class="list-number">$1</span><span>$2</span></div>')
      f = f.replace(/^[-•]\s(.+)$/gm, '<div class="bullet-item">• $1</div>')
      f = f.replace(/\n/g, '<br>')
      return f
    },
    quickAsk(question) {
      this.userInput = question
      this.sendMessage()
    },
    resetChat() {
      this.currentSessionId = null
      this.messages = [
        { sender: 'bot', text: 'Hello! I am the COLRIS Library AI Assistant. How can I help you today?' },
      ]
    },
    toggleVoice() {
      if (!('webkitSpeechRecognition' in window) && !('SpeechRecognition' in window)) {
        alert('Voice input is not supported in your browser. Please use Chrome.')
        return
      }
      if (this.isRecording) {
        this.recognition.stop()
        return
      }
      var SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
      this.recognition = new SpeechRecognition()
      this.recognition.lang = 'en-US'
      this.recognition.continuous = true
      this.recognition.interimResults = false
      this.recognition.maxAlternatives = 1
      this.recognition.onstart = () => { this.isRecording = true }
      this.recognition.onresult = (event) => {
        var transcript = event.results[event.results.length - 1][0].transcript
        this.userInput = transcript
      }
      this.recognition.onerror = (event) => {
        console.error('Speech error:', event.error)
        this.isRecording = false
        if (event.error === 'not-allowed') {
          alert('Microphone access denied. Please allow microphone access in your browser settings.')
        }
      }
      this.recognition.onend = () => {
        this.isRecording = false
        if (this.userInput.trim() !== '') {
          this.$nextTick(() => { this.sendMessage() })
        }
      }
      this.recognition.start()
    },
    async searchBooks() {
      if (!this.searchQuery.trim()) return
      this.isSearching = true
      this.hasSearched = true
      this.searchResults = []
      try {
        var token = localStorage.getItem('token')
        var response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/books/search', {
          params: { query: this.searchQuery },
          headers: { Authorization: 'Bearer ' + token, Accept: 'application/json' },
        })
        this.searchResults = response.data.books || []
      } catch (error) {
        console.error('Search failed:', error)
      } finally {
        this.isSearching = false
      }
    },
    async loadSessionMessages(sessionId) {
      try {
        var token = localStorage.getItem('token')
        var response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/chat/session/' + sessionId, {
          headers: { Authorization: 'Bearer ' + token, Accept: 'application/json' },
        })
        this.currentSessionId = sessionId
        this.messages = response.data.messages.map((msg) => ({
          sender: msg.sender,
          text: msg.message,
        }))
      } catch (error) {
        console.error('Failed to load session:', error)
      }
    },
    async sendMessage() {
      if (this.userInput.trim() === '') return
      var msg = this.userInput.toLowerCase()
      if (msg.indexOf('new arrival') !== -1 || msg.indexOf('latest arrival') !== -1 || msg.indexOf('new book') !== -1 || msg.indexOf('recently added') !== -1) {
        this.messages.push({ sender: 'user', text: this.userInput })
        this.userInput = ''
        this.messages.push({ sender: 'bot', text: 'Sure! Let me show you our latest arrivals now!' })
        setTimeout(() => { this.openLatestArrivals() }, 800)
        return
      }
      this.messages.push({ sender: 'user', text: this.userInput })
      var userText = this.userInput
      this.userInput = ''
      this.isLoading = true
      try {
        var token = localStorage.getItem('token')
        var response = await axios.post(
          'https://colris-chatbot-backend-production.up.railway.app/api/chat',
          { message: userText, session_id: this.currentSessionId },
          { headers: { Authorization: 'Bearer ' + token, Accept: 'application/json' } },
        )
        if (!this.currentSessionId) {
          this.currentSessionId = response.data.session_id
          this.$emit('new-session', this.currentSessionId)
        }
        this.messages.push({ sender: 'bot', text: response.data.reply })
      } catch (error) {
        this.messages.push({ sender: 'bot', text: 'Sorry, I am having trouble connecting. Please try again.' })
      } finally {
        this.isLoading = false
        this.$nextTick(() => {
          if (this.$refs.messageContainer) {
            this.$refs.messageContainer.scrollTop = this.$refs.messageContainer.scrollHeight
          }
        })
      }
    },
  },
}
</script>

<style scoped>
.chat-container { display: flex; flex-direction: column; height: 100vh; background: #faf7f2; font-family: 'Segoe UI', Arial, sans-serif; }
.chat-header { padding: 14px 20px; background: #fff; border-bottom: 2px solid #e8dcc8; display: flex; align-items: center; gap: 16px; box-shadow: 0 2px 12px rgba(92, 61, 46, 0.08); }
.menu-btn { background: transparent; border: none; color: #5c3d2e; font-size: 22px; cursor: pointer; padding: 4px 8px; border-radius: 6px; }
.menu-btn:hover { background: #f5ede0; }
.header-content { display: flex; align-items: center; gap: 12px; flex: 1; }
.header-avatar { font-size: 22px; background: linear-gradient(135deg, #5c3d2e, #8b5e3c); border-radius: 10px; width: 42px; height: 42px; display: flex; align-items: center; justify-content: center; box-shadow: 0 2px 8px rgba(92, 61, 46, 0.2); }
.header-content h2 { margin: 0; font-size: 16px; font-weight: 700; color: #2c1810; font-family: 'Georgia', serif; }
.header-status { color: #8b7355; font-size: 12px; margin: 2px 0 0; display: flex; align-items: center; gap: 6px; }
.status-dot { width: 7px; height: 7px; background: #22c55e; border-radius: 50%; display: inline-block; animation: pulse 2s infinite; }
@keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.4; } }
.search-toggle-btn { background: #5c3d2e; border: none; color: #faf7f2; padding: 8px 18px; border-radius: 6px; cursor: pointer; font-size: 13px; font-weight: 600; white-space: nowrap; transition: all 0.2s; }
.search-toggle-btn:hover { background: #2c1810; }
.announcement-banner { background: linear-gradient(135deg, #2c1810, #5c3d2e); color: #fdf6e3; padding: 10px 24px; font-size: 13px; font-weight: 500; display: flex; align-items: center; justify-content: center; gap: 10px; letter-spacing: 0.3px; border-left: 4px solid #c9a84c; box-shadow: 0 2px 8px rgba(44, 24, 16, 0.15); }
.announcement-dot { width: 6px; height: 6px; background: #c9a84c; border-radius: 50%; flex-shrink: 0; animation: pulse 2s infinite; }
.announcement-label { font-weight: 700; color: #c9a84c; font-size: 11px; text-transform: uppercase; letter-spacing: 1px; flex-shrink: 0; }
.announcement-divider { color: rgba(253, 246, 227, 0.3); flex-shrink: 0; }
.search-panel { flex: 1; overflow-y: auto; padding: 20px; display: flex; flex-direction: column; gap: 16px; background: #faf7f2; }
.search-bar { display: flex; gap: 10px; }
.search-bar input { flex: 1; padding: 12px 18px; border: 1.5px solid #e8dcc8; border-radius: 8px; font-size: 14px; outline: none; background: white; color: #2c1810; }
.search-bar input:focus { border-color: #c9a84c; box-shadow: 0 0 0 3px rgba(201, 168, 76, 0.15); }
.search-btn { background: #5c3d2e; border: none; color: white; padding: 0 20px; border-radius: 8px; font-size: 13px; font-weight: 600; cursor: pointer; height: 48px; }
.search-btn:hover { background: #2c1810; }
.search-loading { display: flex; flex-direction: column; align-items: center; gap: 12px; padding: 40px; color: #8b7355; }
.loading-dots { display: flex; gap: 6px; }
.loading-dots span { width: 10px; height: 10px; background: #c9a84c; border-radius: 50%; animation: bounce 1.2s infinite ease-in-out; }
.loading-dots span:nth-child(2) { animation-delay: 0.2s; }
.loading-dots span:nth-child(3) { animation-delay: 0.4s; }
.no-results { text-align: center; color: #8b7355; padding: 40px; font-size: 16px; }
.book-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 16px; }
.book-card { background: white; border: 1.5px solid #e8dcc8; border-radius: 12px; overflow: hidden; display: flex; flex-direction: column; transition: transform 0.2s, box-shadow 0.2s; }
.book-card:hover { transform: translateY(-4px); box-shadow: 0 8px 24px rgba(92, 61, 46, 0.12); border-color: #c9a84c; }
.book-cover { height: 160px; background: linear-gradient(135deg, #f5ede0, #e8dcc8); display: flex; align-items: center; justify-content: center; overflow: hidden; }
.book-cover img { width: 100%; height: 100%; object-fit: cover; }
.no-cover { font-size: 48px; opacity: 0.3; }
.book-info { padding: 14px; display: flex; flex-direction: column; gap: 6px; flex: 1; }
.source-badge { font-size: 10px; color: #b8a898; margin: 0; text-transform: uppercase; letter-spacing: 1px; }
.book-title { color: #2c1810; font-size: 14px; font-weight: 700; line-height: 1.4; margin: 0; font-family: 'Georgia', serif; }
.book-author { color: #8b5e3c; font-size: 12px; margin: 0; }
.book-year { color: #b8a898; font-size: 11px; margin: 0; }
.book-subjects { display: flex; flex-wrap: wrap; gap: 4px; margin-top: 4px; }
.subject-tag { background: #fdf6e3; border: 1px solid #e8dcc8; color: #8b5e3c; font-size: 10px; padding: 2px 8px; border-radius: 6px; }
.book-actions { display: flex; gap: 8px; margin-top: 8px; flex-wrap: wrap; align-items: center; }
.view-btn { background: #5c3d2e; color: white; text-decoration: none; padding: 6px 12px; border-radius: 6px; font-size: 11px; font-weight: 600; }
.view-btn:hover { opacity: 0.85; }
.download-btn { background: #c9a84c; color: white; text-decoration: none; padding: 6px 12px; border-radius: 6px; font-size: 11px; font-weight: 600; }
.access-badge { padding: 4px 10px; border-radius: 6px; font-size: 11px; font-weight: 500; }
.access-green { background: #f0fdf4; border: 1px solid #bbf7d0; color: #16a34a; }
.access-blue { background: #fdf6e3; border: 1px solid #e8dcc8; color: #8b5e3c; }
.access-gray { background: #f9f6f0; border: 1px solid #e8dcc8; color: #8b7355; }
.arrivals-header { display: flex; align-items: flex-start; justify-content: space-between; flex-wrap: wrap; gap: 12px; padding: 4px 0 16px; border-bottom: 1.5px solid #e8dcc8; margin-bottom: 16px; }
.arrivals-title { color: #2c1810; font-size: 20px; font-weight: 800; margin: 0; font-family: 'Georgia', serif; }
.arrivals-sub { color: #8b7355; font-size: 13px; margin: 4px 0 0; }
.arrivals-actions { display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
.clr-link-btn { background: #c9a84c; color: white; text-decoration: none; padding: 8px 16px; border-radius: 6px; font-size: 12px; font-weight: 600; white-space: nowrap; }
.clr-link-btn:hover { opacity: 0.85; }
.close-arrivals-btn { background: transparent; border: 1.5px solid #e8dcc8; color: #5c3d2e; padding: 8px 14px; border-radius: 6px; cursor: pointer; font-size: 12px; font-weight: 600; }
.close-arrivals-btn:hover { background: #f5ede0; }
.arrivals-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(260px, 1fr)); gap: 16px; }
.arrival-card { background: white; border: 1.5px solid #e8dcc8; border-radius: 12px; padding: 18px; position: relative; box-shadow: 0 2px 8px rgba(92, 61, 46, 0.06); transition: transform 0.2s, box-shadow 0.2s; display: flex; flex-direction: column; gap: 8px; }
.arrival-card:hover { transform: translateY(-3px); box-shadow: 0 8px 24px rgba(92, 61, 46, 0.12); border-color: #c9a84c; }
.arrival-badge { position: absolute; top: 12px; right: 12px; background: #c9a84c; color: white; font-size: 10px; font-weight: 700; padding: 3px 8px; border-radius: 20px; letter-spacing: 1px; }
.arrival-icon { font-size: 32px; }
.arrival-title { color: #2c1810; font-size: 15px; font-weight: 700; margin: 0; font-family: 'Georgia', serif; line-height: 1.4; }
.arrival-author { color: #8b5e3c; font-size: 13px; margin: 0; }
.arrival-category { display: inline-block; background: #fdf6e3; color: #8b5e3c; border: 1px solid #e8dcc8; padding: 2px 10px; border-radius: 20px; font-size: 11px; font-weight: 600; }
.arrival-meta { display: flex; flex-direction: column; gap: 4px; margin-top: 4px; }
.arrival-location { color: #8b7355; font-size: 12px; }
.arrival-stock { font-size: 12px; font-weight: 600; }
.in-stock { color: #16a34a; }
.out-stock { color: #dc2626; }
.arrival-colris-btn { display: inline-block; margin-top: 8px; background: #5c3d2e; color: white; text-decoration: none; padding: 8px 14px; border-radius: 6px; font-size: 12px; font-weight: 600; text-align: center; }
.arrival-colris-btn:hover { background: #2c1810; }
.chat-messages { flex: 1; overflow-y: auto; padding: 24px 20px; display: flex; flex-direction: column; gap: 16px; background: #faf7f2; }
.chat-messages::-webkit-scrollbar { width: 4px; }
.chat-messages::-webkit-scrollbar-thumb { background: #e8dcc8; border-radius: 4px; }
.welcome-banner { text-align: center; padding: 32px 20px; margin-bottom: 10px; }
.welcome-icon { font-size: 48px; margin-bottom: 16px; }
.welcome-banner h3 { color: #2c1810; font-size: 22px; margin-bottom: 8px; font-weight: 700; font-family: 'Georgia', serif; }
.welcome-sub { color: #8b7355; font-size: 14px; margin-bottom: 24px; }
.quick-actions { display: flex; flex-wrap: wrap; gap: 10px; justify-content: center; }
.quick-actions button { background: white; border: 1.5px solid #e8dcc8; color: #5c3d2e; padding: 8px 16px; border-radius: 6px; cursor: pointer; font-size: 13px; font-weight: 500; transition: all 0.2s; }
.quick-actions button:hover { border-color: #c9a84c; color: #2c1810; background: #fdf6e3; }
.arrivals-quick-btn { background: linear-gradient(135deg, #c9a84c, #b8960a) !important; color: white !important; border-color: #c9a84c !important; }
.message { display: flex; align-items: flex-start; gap: 10px; }
.message.user { flex-direction: row-reverse; }
.message-avatar { font-size: 16px; width: 34px; height: 34px; border-radius: 8px; background: linear-gradient(135deg, #5c3d2e, #8b5e3c); display: flex; align-items: center; justify-content: center; flex-shrink: 0; margin-top: 4px; }
.user-avatar { background: linear-gradient(135deg, #c9a84c, #b8960a); }
.message-bubble { max-width: 70%; padding: 12px 16px; border-radius: 12px; font-size: 14px; line-height: 1.7; }
.message.user .message-bubble { background: #5c3d2e; color: #faf7f2; border-bottom-right-radius: 4px; }
.message.bot .message-bubble { background: white; color: #2c1810; border-bottom-left-radius: 4px; border: 1.5px solid #e8dcc8; box-shadow: 0 1px 4px rgba(92, 61, 46, 0.06); }
.message-bubble :deep(a) { color: #8b5e3c; text-decoration: underline; word-break: break-all; }
.message-bubble :deep(strong) { color: #2c1810; font-weight: 700; }
.message-bubble :deep(.list-item) { display: flex; gap: 10px; margin: 8px 0; padding: 8px 12px; background: #fdf6e3; border-radius: 8px; border-left: 3px solid #c9a84c; }
.message-bubble :deep(.list-number) { background: #5c3d2e; color: white; width: 22px; height: 22px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 11px; font-weight: 700; flex-shrink: 0; }
.message-bubble :deep(.bullet-item) { margin: 6px 0; padding-left: 8px; color: #2c1810; }
.typing-indicator { display: flex; align-items: center; gap: 4px; padding: 12px 16px !important; }
.typing-indicator span { width: 8px; height: 8px; background: #c9a84c; border-radius: 50%; animation: bounce 1.2s infinite ease-in-out; }
.typing-indicator span:nth-child(2) { animation-delay: 0.2s; }
.typing-indicator span:nth-child(3) { animation-delay: 0.4s; }
@keyframes bounce { 0%, 60%, 100% { transform: translateY(0); } 30% { transform: translateY(-8px); } }
.chat-input { display: flex; padding: 16px 20px; background: white; border-top: 2px solid #e8dcc8; gap: 12px; align-items: center; box-shadow: 0 -2px 12px rgba(92, 61, 46, 0.06); }
.chat-input input { flex: 1; padding: 12px 18px; border: 1.5px solid #e8dcc8; border-radius: 8px; font-size: 14px; outline: none; background: #faf7f2; color: #2c1810; font-family: 'Segoe UI', Arial, sans-serif; }
.chat-input input:focus { border-color: #c9a84c; background: white; box-shadow: 0 0 0 3px rgba(201, 168, 76, 0.15); }
.send-btn { width: 44px; height: 44px; background: #5c3d2e; color: white; border: none; border-radius: 8px; font-size: 16px; cursor: pointer; display: flex; align-items: center; justify-content: center; transition: transform 0.2s, background 0.2s; flex-shrink: 0; }
.send-btn:hover { background: #2c1810; transform: scale(1.05); }
.send-btn:disabled { opacity: 0.5; cursor: not-allowed; transform: none; }
.mic-btn { width: 44px; height: 44px; background: #faf7f2; border: 1.5px solid #e8dcc8; border-radius: 8px; font-size: 18px; cursor: pointer; display: flex; align-items: center; justify-content: center; flex-shrink: 0; transition: all 0.2s; }
.mic-btn:hover { background: #f5ede0; border-color: #c9a84c; }
.mic-btn.recording { background: #dc2626; border-color: #dc2626; animation: micPulse 1s infinite; }
.arrivals-btn { width: 44px; height: 44px; background: #fdf6e3; border: 1.5px solid #c9a84c; border-radius: 8px; font-size: 18px; cursor: pointer; display: flex; align-items: center; justify-content: center; flex-shrink: 0; transition: all 0.2s; }
.arrivals-btn:hover { background: #c9a84c; }
@keyframes micPulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.1); } }
</style>
