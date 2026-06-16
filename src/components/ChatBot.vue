<template>
  <div class="chat-container">
    <div class="chat-header">

      <div class="header-content">
        <div class="header-avatar">📚</div>
        <div>
          <h2>COLRIS Library Assistant</h2>
          <p class="header-status">
            <span class="status-dot"></span> Online • Welcome, {{ user && user.name ? user.name : 'Student' }}
          </p>
        </div>
      </div>
      <button @click="showSaved = !showSaved" class="save-toggle-btn" title="Saved Answers">🔖</button>
      <button @click="showSearch = !showSearch; showArrivals = false" class="search-toggle-btn">
        {{ showSearch ? 'Chat' : 'Search Books' }}
      </button>
    </div>

    <div v-if="showSaved" class="saved-panel">
      <div class="saved-header">
        <span>🔖 Saved Answers ({{ savedAnswers.length }})</span>
        <button @click="showSaved = false" class="close-saved">✕</button>
      </div>
      <div v-if="savedAnswers.length === 0" class="saved-empty">No saved answers yet. Click 🔖 on any response to save it.</div>
      <div v-for="(ans, i) in savedAnswers" :key="i" class="saved-item">
        <p class="saved-text" v-html="formatMessage(ans.text)"></p>
        <div class="saved-meta">
          <span>{{ ans.time }}</span>
          <button @click="savedAnswers.splice(i, 1)" class="delete-saved">Remove</button>
        </div>
      </div>
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
        <h3>Welcome to COLRIS Library Assistant</h3>
        <p class="welcome-sub">Your 24/7 Covenant University Library companion — ask me anything</p>
        <div class="quick-chips">
          <button class="chip" @click="quickAsk('What are the library opening hours?')">🕐 Hours</button>
          <button class="chip" @click="quickAsk('How do I borrow a book?')">📖 Borrowing</button>
          <button class="chip" @click="quickAsk('What is the fine for returning a book late?')">💰 Fines</button>
          <button class="chip" @click="quickAsk('What databases are available?')">🗄️ Databases</button>
          <button class="chip" @click="quickAsk('How do I find a book in COLRIS?')">🔍 Find a Book</button>
          <button class="chip" @click="quickAsk('What are the library rules?')">📋 Rules</button>
          <button class="chip arrivals-chip" @click="openLatestArrivals">🆕 New Arrivals</button>
        </div>

      </div>
      <div v-for="(message, index) in messages" :key="index" :class="['message', message.sender]">
        <div class="message-avatar" v-if="message.sender === 'bot'">📚</div>
        <div style="display:flex;flex-direction:column;gap:8px;max-width:85%;">
          <div v-if="message.attachment" class="attachment-card">
            <div class="attachment-icon">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
            </div>
            <div class="attachment-info">
              <div class="attachment-name">{{ message.attachment.name }}</div>
              <div class="attachment-type">{{ message.attachment.type || 'Document' }}</div>
            </div>
          </div>
          <div class="message-bubble" v-if="message.text" v-html="formatMessage(message.text)"></div>
          <div v-if="message.sender === 'bot' && !message.isFallback" class="message-actions">
            <button @click="thumbs(index, 'up')" :class="['action-btn', message.feedback === 'up' ? 'active-up' : '']" title="Helpful">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 9V5a3 3 0 0 0-3-3l-4 9v11h11.28a2 2 0 0 0 2-1.7l1.38-9a2 2 0 0 0-2-2.3H14z"/><path d="M7 22H4a2 2 0 0 1-2-2v-7a2 2 0 0 1 2-2h3"/></svg>
            </button>
            <button @click="thumbs(index, 'down')" :class="['action-btn', message.feedback === 'down' ? 'active-down' : '']" title="Not helpful">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M10 15v4a3 3 0 0 0 3 3l4-9V2H5.72a2 2 0 0 0-2 1.7l-1.38 9a2 2 0 0 0 2 2.3H10z"/><path d="M17 2h2.67A2.31 2.31 0 0 1 22 4v7a2.31 2.31 0 0 1-2.33 2H17"/></svg>
            </button>
            <button @click="saveAnswer(message.text)" class="action-btn" title="Save answer">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z"/></svg>
            </button>
            <button v-if="message.text && message.text.includes('colris.covenantuniversity.edu.ng')" @click="shareLink(message.text)" class="action-btn" title="Copy COLRIS link">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="13" height="13" rx="2" ry="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg>
            </button>
          </div>
          <div v-if="message.isFallback" class="handoff-card">
            <p class="handoff-title">🧑‍💼 Need more help?</p>
            <p class="handoff-sub">A librarian can assist you directly.</p>
            <div class="handoff-actions">
              <a href="mailto:library@covenantuniversity.edu.ng?subject=Library%20Enquiry&body=Hello%2C%20I%20need%20help%20with%20the%20following%3A%20" class="handoff-btn primary">✉️ Contact a Librarian</a>
            </div>
            <div class="handoff-suggestions">
              <p class="suggestions-label">Or try one of these:</p>
              <div class="suggestion-chips">
                <button @click="quickAsk('What are the library opening hours?')" class="chip">🕐 Library Hours</button>
                <button @click="quickAsk('How do I borrow a book?')" class="chip">📖 Borrowing</button>
                <button @click="quickAsk('What is the fine for late returns?')" class="chip">💰 Fines</button>
                <button @click="quickAsk('What databases are available?')" class="chip">🗄️ Databases</button>
              </div>
            </div>
          </div>
        </div>
        <div class="message-avatar user-avatar" v-if="message.sender === 'user'">👤</div>
      </div>
      <div v-if="isLoading" class="message bot">
        <div class="message-avatar">📚</div>
        <div class="message-bubble typing-indicator"><span></span><span></span><span></span></div>
      </div>
    </div>

    <div v-if="!showSearch && !showArrivals" class="chat-input-area">
      <div v-if="attachedFile" class="file-preview-bar">
        <div class="file-preview-card">
          <img v-if="attachedFile.type.startsWith('image')" :src="attachedPreviewUrl" class="file-preview-img" />
          <div v-else class="file-preview-icon">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
          </div>
          <div class="file-preview-info">
            <div class="file-preview-name">{{ attachedFile.name }}</div>
            <div class="file-preview-size">{{ (attachedFile.size / 1024).toFixed(1) }} KB</div>
          </div>
          <button @click="removeFile" class="file-preview-remove">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
          </button>
        </div>
      </div>
      <div class="chat-input">
        <button @click="toggleVoice" class="mic-btn" :class="{ recording: isRecording }">
          <span v-if="!isRecording">🎤</span>
          <span v-else>⏹</span>
        </button>
        <button @click="$refs.fileInput.click()" class="attach-btn" title="Attach file">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21.44 11.05l-9.19 9.19a6 6 0 0 1-8.49-8.49l9.19-9.19a4 4 0 0 1 5.66 5.66l-9.2 9.19a2 2 0 0 1-2.83-2.83l8.49-8.48"/></svg>
        </button>
        <input ref="fileInput" type="file" accept=".pdf,.png,.jpg,.jpeg,.txt,.doc,.docx" style="display:none" @change="handleFileUpload" />
        <div v-if="attachedFile" class="attached-file">
          <span>📎 {{ attachedFile.name }}</span>
          <button @click="attachedFile = null; attachedContent = ''" class="remove-file">✕</button>
        </div>
        <input v-model="userInput" @keyup.enter="sendMessage" placeholder="Ask about books, hours, fines, databases, or anything else..." type="text" :disabled="isLoading" />
        <button @click="sendMessage" :disabled="isLoading" class="send-btn">
          <span v-if="!isLoading">➤</span>
          <span v-else>...</span>
        </button>
      </div>
    </div>
    <!-- Book Request Modal -->
    <div v-if="showBookRequest" class="modal-overlay" @click.self="showBookRequest = false">
      <div class="feature-modal">
        <div class="modal-header">
          <h3>📚 Request a Book</h3>
          <button @click="showBookRequest = false" class="modal-close">✕</button>
        </div>
        <p class="modal-sub">Can't find a book in our library? Submit a request and the librarian will review it.</p>
        <div class="modal-form">
          <div class="form-group">
            <label>Book Title *</label>
            <input v-model="bookReq.title" type="text" placeholder="e.g. Introduction to Algorithms" />
          </div>
          <div class="form-group">
            <label>Author (optional)</label>
            <input v-model="bookReq.author" type="text" placeholder="e.g. Thomas H. Cormen" />
          </div>
          <div class="form-group">
            <label>Why do you need this book?</label>
            <textarea v-model="bookReq.reason" placeholder="e.g. For my final year project on machine learning" rows="3"></textarea>
          </div>
          <div v-if="bookReqMsg" :class="['req-msg', bookReqSuccess ? 'success' : 'error']">{{ bookReqMsg }}</div>
          <button @click="submitBookRequest" class="modal-submit-btn" :disabled="isSubmittingReq">
            {{ isSubmittingReq ? 'Submitting...' : 'Submit Request' }}
          </button>
        </div>
      </div>
    </div>

    <!-- Citation Generator Modal -->
    <div v-if="showCitation" class="modal-overlay" @click.self="showCitation = false">
      <div class="feature-modal">
        <div class="modal-header">
          <h3>📝 Citation Generator</h3>
          <button @click="showCitation = false" class="modal-close">✕</button>
        </div>
        <p class="modal-sub">Generate a properly formatted citation for any book or resource.</p>
        <div class="modal-form">
          <div class="form-group">
            <label>Author(s)</label>
            <input v-model="citForm.author" type="text" placeholder="e.g. Smith, J., and Jones, A." />
          </div>
          <div class="form-group">
            <label>Title of Book/Article</label>
            <input v-model="citForm.title" type="text" placeholder="e.g. Introduction to Computing" />
          </div>
          <div class="form-group">
            <label>Year of Publication</label>
            <input v-model="citForm.year" type="text" placeholder="e.g. 2023" />
          </div>
          <div class="form-group">
            <label>Publisher</label>
            <input v-model="citForm.publisher" type="text" placeholder="e.g. Pearson Education" />
          </div>
          <div class="form-group">
            <label>City of Publication</label>
            <input v-model="citForm.city" type="text" placeholder="e.g. New York" />
          </div>
          <div class="form-group">
            <label>Citation Style</label>
            <select v-model="citForm.style">
              <option value="apa">APA</option>
              <option value="mla">MLA</option>
              <option value="harvard">Harvard</option>
            </select>
          </div>
          <div v-if="citResult" class="citation-result">
            <p class="cit-label">Generated Citation:</p>
            <p class="cit-text">{{ citResult }}</p>
            <button @click="copyCitation" class="copy-cit-btn">{{ citCopied ? "Copied!" : "Copy" }}</button>
          </div>
          <button @click="generateCitation" class="modal-submit-btn">Generate Citation</button>
        </div>
      </div>
    </div>

    <!-- My Book Requests Modal -->
    <div v-if="showMyRequests" class="modal-overlay" @click.self="showMyRequests = false">
      <div class="feature-modal">
        <div class="modal-header">
          <h3>📋 My Book Requests</h3>
          <button @click="showMyRequests = false" class="modal-close">✕</button>
        </div>
        <div v-if="myRequests.length === 0" class="empty-requests">No book requests yet.</div>
        <div v-for="req in myRequests" :key="req.id" class="req-item">
          <div class="req-title">{{ req.title }}</div>
          <div class="req-author" v-if="req.author">by {{ req.author }}</div>
          <div class="req-status-row">
            <span :class="['req-badge', req.status]">{{ req.status }}</span>
            <span class="req-date">{{ new Date(req.created_at).toLocaleDateString() }}</span>
          </div>
          <div v-if="req.admin_note" class="req-note">Note: {{ req.admin_note }}</div>
        </div>
      </div>
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
      showSaved: false,
      attachedFile: null,
      attachedContent: '',
      attachedPreviewUrl: '',
      savedAnswers: [],
      showBookRequest: false,
      showCitation: false,
      showMyRequests: false,
      myRequests: [],
      bookReq: { title: '', author: '', reason: '' },
      bookReqMsg: '',
      bookReqSuccess: false,
      isSubmittingReq: false,
      citForm: { author: '', title: '', year: '', publisher: '', city: '', style: 'apa' },
      citResult: '',
      citCopied: false,
      didYouKnowTips: [
        'You can ask me to find books by topic, author, or title.',
        'Type "library hours" to get today\'s opening times instantly.',
        'You can ask me what the borrowing limit is for students.',
        'Ask me about available databases and e-journals.',
        'You can also ask general questions — not just library ones!',
        'Type "new arrivals" to see the latest books in the library.',
        'Ask me about fines for late book returns.',
        'You can ask me about library rules and regulations.',
      ],
      didYouKnowTip: '',
      searchQuery: '',
      searchResults: [],
      isSearching: false,
      hasSearched: false,
      announcement: '',
      newArrivals: [],
      isLoadingArrivals: false,
      messages: [],
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
    // Random Did You Know tip
    this.didYouKnowTip = this.didYouKnowTips[Math.floor(Math.random() * this.didYouKnowTips.length)]
    // Personalised greeting
    const userName = this.user && this.user.name ? this.user.name.split(' ')[0] : 'there'
    this.messages = [
      { sender: 'bot', text: 'Welcome back, ' + userName + '! 👋 I am the COLRIS Library AI Assistant. How can I help you today?' }
    ]
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
    async thumbs(index, type) {
      const msg = this.messages[index]
      const newFeedback = msg.feedback === type ? null : type
      this.messages[index] = Object.assign({}, msg, { feedback: newFeedback })
      this.messages = [...this.messages]
      if (newFeedback) {
        try {
          const token = localStorage.getItem('token')
          await fetch('https://colris-chatbot-backend-production.up.railway.app/api/chat/feedback', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json', 'Authorization': 'Bearer ' + token },
            body: JSON.stringify({ session_id: this.currentSessionId, message_index: index, feedback: newFeedback })
          })
        } catch (e) { console.error(e) }
      }
    },
    saveAnswer(text) {
      const now = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
      this.savedAnswers.unshift({ text: text, time: now })
      this.showSaved = true
    },
    async handleFileUpload(event) {
      const file = event.target.files[0]
      if (!file) return
      this.attachedFile = file
      const reader = new FileReader()
      reader.onload = (e) => {
        if (file.type === 'application/pdf' || file.name.endsWith('.pdf')) {
          this.attachedContent = '[PDF FILE: ' + file.name + '] The user has uploaded a PDF. Please acknowledge you received it and ask what they need help with regarding this document.'
        } else {
          this.attachedContent = e.target.result
        }
      }
      if (file.type.startsWith('text') || file.name.endsWith('.txt')) {
        reader.readAsText(file)
      } else {
        reader.readAsDataURL(file)
      }
      this.messages.push({ sender: 'bot', text: '📎 File attached: **' + file.name + '**\n\nWhat would you like to know about this file?', isFallback: false })
    },
    async openBookRequest() {
      this.messages.push({ sender: 'user', text: 'Request a Book' })
      this.messages.push({
        sender: 'bot',
        text: '📚 **Request a Book**\n\nType your request in the chat box below using this format:\n\nREQUEST: Book Title by Author - Reason you need it\n\nExample: REQUEST: Clean Code by Robert Martin - For my software engineering project',
        isFallback: false
      })
      this.$nextTick(() => {
        if (this.$refs.messageContainer) this.$refs.messageContainer.scrollTop = 999999
      })
    },
    openCitation() {
      this.messages.push({ sender: 'user', text: 'Citation Generator' })
      this.messages.push({
        sender: 'bot',
        text: '📝 **Citation Generator**\n\nType your book details in the chat box using this format:\n\nCITE: Author | Title | Year | Publisher | Style\n\nStyle can be APA, MLA, or Harvard\nExample: CITE: Smith, J. | Introduction to Computing | 2023 | Pearson | APA',
        isFallback: false
      })
      this.$nextTick(() => {
        if (this.$refs.messageContainer) this.$refs.messageContainer.scrollTop = 999999
      })
    },
    async openMyRequests() {
      this.messages.push({ sender: 'user', text: 'My Book Requests' })
      try {
        const token = localStorage.getItem('token')
        const res = await fetch('https://colris-chatbot-backend-production.up.railway.app/api/book-requests/my', {
          headers: { 'Authorization': 'Bearer ' + token, 'Accept': 'application/json' }
        })
        const data = await res.json()
        const reqs = data.requests || []
        if (reqs.length === 0) {
          this.messages.push({ sender: 'bot', text: 'You have no book requests yet. Click <strong>Request Book</strong> to submit one!', isFallback: false })
        } else {
          let text = '📋 **Your Book Requests:**\n\n'
          reqs.forEach(function(r, i) {
            text += (i+1) + '. **' + r.title + '**'
            if (r.author) text += ' by ' + r.author
            text += ' — ' + r.status.toUpperCase()
            if (r.admin_note) text += ' (Note: ' + r.admin_note + ')'
            text += '\n'
          })
          this.messages.push({ sender: 'bot', text: text, isFallback: false })
        }
        this.$nextTick(() => {
          if (this.$refs.messageContainer) this.$refs.messageContainer.scrollTop = 999999
        })
      } catch (e) {
        this.messages.push({ sender: 'bot', text: 'Could not load your requests. Please try again.', isFallback: false })
      }
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
      if (this.userInput.trim().toUpperCase().startsWith('REQUEST:')) {
        const parts = this.userInput.replace(/^REQUEST:/i, '').trim()
        this.messages.push({ sender: 'user', text: this.userInput })
        this.userInput = ''
        try {
          const token = localStorage.getItem('token')
          const byParts = parts.split(' by ')
          const title = byParts[0].trim()
          const rest = byParts[1] || ''
          const author = rest.split(' - ')[0].trim()
          const reason = rest.split(' - ')[1] ? rest.split(' - ')[1].trim() : ''
          const res = await fetch('https://colris-chatbot-backend-production.up.railway.app/api/book-requests', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json', 'Authorization': 'Bearer ' + token },
            body: JSON.stringify({ title: title, author: author, reason: reason })
          })
          if (res.ok) {
            this.messages.push({ sender: 'bot', text: 'Your book request for **' + title + '** has been submitted! The librarian will review it.', isFallback: false })
          } else {
            this.messages.push({ sender: 'bot', text: 'Could not submit request. Please try again.', isFallback: false })
          }
        } catch (e) {
          this.messages.push({ sender: 'bot', text: 'Network error. Please try again.', isFallback: false })
        }
        return
      }
      if (this.userInput.trim().toUpperCase().startsWith('CITE:')) {
        const parts = this.userInput.replace(/^CITE:/i, '').trim().split('|').map(function(p) { return p.trim() })
        const author = parts[0] || ''
        const title = parts[1] || ''
        const year = parts[2] || ''
        const publisher = parts[3] || ''
        const style = (parts[4] || 'APA').toUpperCase()
        this.messages.push({ sender: 'user', text: this.userInput })
        this.userInput = ''
        let citation = ''
        if (style === 'APA') citation = author + ' (' + year + '). ' + title + '. ' + publisher + '.'
        else if (style === 'MLA') citation = author + '. "' + title + '." ' + publisher + ', ' + year + '.'
        else citation = author + ' ' + year + ', ' + title + ', ' + publisher + '.'
        this.messages.push({ sender: 'bot', text: '📝 **' + style + ' Citation:**\n\n' + citation, isFallback: false })
        return
      }
      // Include attached file content in message if present
      if (this.attachedContent) {
        var fileContext = '\n\n[ATTACHED FILE: ' + (this.attachedFile ? this.attachedFile.name : 'file') + ']\n' + this.attachedContent.substring(0, 3000)
        var originalInput = this.userInput
        this.userInput = originalInput + fileContext
      }
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
      this.attachedFile = null
      this.attachedContent = ''
      this.attachedPreviewUrl = ''
      if (this.$refs.fileInput) this.$refs.fileInput.value = ''
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
        this.messages.push({ sender: 'bot', text: response.data.reply, isFallback: response.data.is_fallback || false })
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
* { box-sizing: border-box; }

.chat-container { display: flex; flex-direction: column; height: 100vh; background: #f5f0e8; font-family: 'Segoe UI', Arial, sans-serif; animation: fadeIn 0.4s ease; }
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

/* ── HEADER ── */
.chat-header { padding: 0 20px; background: #1a0f0a; display: flex; align-items: center; gap: 14px; box-shadow: 0 2px 16px rgba(0,0,0,0.3); min-height: 58px; position: relative; z-index: 10; }
.menu-btn { background: transparent; border: none; color: rgba(255,255,255,0.6); font-size: 20px; cursor: pointer; padding: 6px 8px; border-radius: 6px; transition: all 0.2s; flex-shrink: 0; }
.menu-btn:hover { background: rgba(255,255,255,0.08); color: white; }
.header-content { display: flex; align-items: center; gap: 12px; flex: 1; }
.header-avatar { font-size: 20px; background: rgba(201,168,76,0.2); border-radius: 10px; width: 38px; height: 38px; display: flex; align-items: center; justify-content: center; border: 1px solid rgba(201,168,76,0.3); flex-shrink: 0; transition: transform 0.3s; }
.header-avatar:hover { transform: rotate(-5deg) scale(1.1); }
.header-content h2 { margin: 0; font-size: 15px; font-weight: 700; color: #ffffff; letter-spacing: 0.3px; }
.header-status { color: rgba(255,255,255,0.5); font-size: 11px; margin: 2px 0 0; display: flex; align-items: center; gap: 6px; }
.status-dot { width: 6px; height: 6px; background: #22c55e; border-radius: 50%; display: inline-block; animation: pulse 2s infinite; }
@keyframes pulse { 0%, 100% { opacity: 1; box-shadow: 0 0 0 0 rgba(34,197,94,0.4); } 50% { opacity: 0.8; box-shadow: 0 0 0 4px rgba(34,197,94,0); } }
.search-toggle-btn { background: rgba(201,168,76,0.15); border: 1px solid rgba(201,168,76,0.35); color: #c9a84c; padding: 7px 16px; border-radius: 6px; cursor: pointer; font-size: 12px; font-weight: 700; white-space: nowrap; transition: all 0.25s; letter-spacing: 0.4px; text-transform: uppercase; }
.search-toggle-btn:hover { background: #c9a84c; color: #1a0f0a; }

/* ── ANNOUNCEMENT ── */
.announcement-banner { background: linear-gradient(90deg, #2c1810, #5c3d2e); color: #fdf6e3; padding: 9px 24px; font-size: 12px; font-weight: 500; display: flex; align-items: center; justify-content: center; gap: 10px; border-left: 3px solid #c9a84c; }
.announcement-dot { width: 5px; height: 5px; background: #c9a84c; border-radius: 50%; flex-shrink: 0; animation: pulse 2s infinite; }
.announcement-label { font-weight: 800; color: #c9a84c; font-size: 10px; text-transform: uppercase; letter-spacing: 1.5px; flex-shrink: 0; }
.announcement-divider { color: rgba(253,246,227,0.3); flex-shrink: 0; }

/* ── SEARCH / ARRIVALS PANEL ── */
.search-panel { flex: 1; overflow-y: auto; padding: 24px; display: flex; flex-direction: column; gap: 16px; background: #f5f0e8; }
.search-bar { display: flex; gap: 10px; }
.search-bar input { flex: 1; padding: 12px 18px; border: 1.5px solid #e2d8cc; border-radius: 10px; font-size: 14px; outline: none; background: white; color: #1a0f0a; transition: all 0.25s; box-shadow: 0 1px 4px rgba(0,0,0,0.06); }
.search-bar input:focus { border-color: #c9a84c; box-shadow: 0 0 0 3px rgba(201,168,76,0.12); }
.search-btn { background: #1a0f0a; border: none; color: white; padding: 0 24px; border-radius: 10px; font-size: 13px; font-weight: 700; cursor: pointer; height: 48px; transition: all 0.25s; letter-spacing: 0.4px; text-transform: uppercase; box-shadow: 0 2px 8px rgba(0,0,0,0.2); }
.search-btn:hover { background: #2c1810; transform: translateY(-1px); box-shadow: 0 4px 16px rgba(0,0,0,0.25); }
.search-loading { display: flex; flex-direction: column; align-items: center; gap: 12px; padding: 60px; color: #8b7355; }
.loading-dots { display: flex; gap: 6px; }
.loading-dots span { width: 10px; height: 10px; background: #c9a84c; border-radius: 50%; animation: bounce 1.2s infinite ease-in-out; }
.loading-dots span:nth-child(2) { animation-delay: 0.2s; }
.loading-dots span:nth-child(3) { animation-delay: 0.4s; }
.no-results { text-align: center; color: #8b7355; padding: 60px; font-size: 15px; }
.book-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 16px; }
.book-card { background: white; border: 1px solid rgba(0,0,0,0.07); border-radius: 14px; overflow: hidden; display: flex; flex-direction: column; transition: all 0.3s ease; box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
.book-card:hover { transform: translateY(-5px); box-shadow: 0 12px 32px rgba(0,0,0,0.1); border-color: #c9a84c; }
.book-cover { height: 160px; background: linear-gradient(135deg, #f5ede0, #e8dcc8); display: flex; align-items: center; justify-content: center; overflow: hidden; }
.book-cover img { width: 100%; height: 100%; object-fit: cover; }
.no-cover { font-size: 48px; opacity: 0.3; }
.book-info { padding: 16px; display: flex; flex-direction: column; gap: 6px; flex: 1; }
.source-badge { font-size: 10px; color: #b8a898; margin: 0; text-transform: uppercase; letter-spacing: 1.5px; font-weight: 700; }
.book-title { color: #1a0f0a; font-size: 14px; font-weight: 700; line-height: 1.4; margin: 0; }
.book-author { color: #8b5e3c; font-size: 12px; margin: 0; }
.book-year { color: #b8a898; font-size: 11px; margin: 0; }
.book-subjects { display: flex; flex-wrap: wrap; gap: 4px; margin-top: 4px; }
.subject-tag { background: #fdf6e3; border: 1px solid #e8dcc8; color: #8b5e3c; font-size: 10px; padding: 2px 8px; border-radius: 4px; font-weight: 600; }
.book-actions { display: flex; gap: 8px; margin-top: 8px; flex-wrap: wrap; align-items: center; }
.view-btn { background: #1a0f0a; color: white; text-decoration: none; padding: 6px 14px; border-radius: 6px; font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.4px; transition: all 0.2s; }
.view-btn:hover { background: #2c1810; transform: translateY(-1px); }
.download-btn { background: #c9a84c; color: white; text-decoration: none; padding: 6px 14px; border-radius: 6px; font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.4px; transition: all 0.2s; }
.download-btn:hover { background: #b8960a; }
.access-badge { padding: 4px 10px; border-radius: 4px; font-size: 10px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.5px; }
.access-green { background: #f0fdf4; border: 1px solid #bbf7d0; color: #16a34a; }
.access-blue { background: #fdf6e3; border: 1px solid #e8dcc8; color: #8b5e3c; }
.access-gray { background: #f5f0e8; border: 1px solid #e8dcc8; color: #8b7355; }

/* ── ARRIVALS ── */
.arrivals-header { display: flex; align-items: flex-start; justify-content: space-between; flex-wrap: wrap; gap: 12px; padding-bottom: 18px; border-bottom: 1px solid #e2d8cc; margin-bottom: 20px; }
.arrivals-title { color: #1a0f0a; font-size: 20px; font-weight: 800; margin: 0; }
.arrivals-sub { color: #8b7355; font-size: 13px; margin: 4px 0 0; }
.arrivals-actions { display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
.clr-link-btn { background: #c9a84c; color: white; text-decoration: none; padding: 8px 18px; border-radius: 6px; font-size: 12px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.4px; transition: all 0.25s; }
.clr-link-btn:hover { background: #b8960a; transform: translateY(-1px); }
.close-arrivals-btn { background: white; border: 1.5px solid #e2d8cc; color: #5c3d2e; padding: 8px 16px; border-radius: 6px; cursor: pointer; font-size: 12px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.4px; transition: all 0.2s; }
.close-arrivals-btn:hover { border-color: #c9a84c; color: #1a0f0a; }
.arrivals-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(260px, 1fr)); gap: 16px; }
.arrival-card { background: white; border: 1px solid rgba(0,0,0,0.07); border-radius: 14px; padding: 20px; position: relative; box-shadow: 0 2px 8px rgba(0,0,0,0.05); transition: all 0.3s ease; display: flex; flex-direction: column; gap: 8px; }
.arrival-card:hover { transform: translateY(-4px); box-shadow: 0 12px 32px rgba(0,0,0,0.1); border-color: #c9a84c; }
.arrival-badge { position: absolute; top: 14px; right: 14px; background: #1a0f0a; color: #c9a84c; font-size: 9px; font-weight: 800; padding: 3px 8px; border-radius: 4px; letter-spacing: 1.5px; text-transform: uppercase; }
.arrival-icon { font-size: 28px; }
.arrival-title { color: #1a0f0a; font-size: 15px; font-weight: 700; margin: 0; line-height: 1.4; }
.arrival-author { color: #8b5e3c; font-size: 13px; margin: 0; }
.arrival-category { display: inline-block; background: #fdf6e3; color: #8b5e3c; border: 1px solid #e8dcc8; padding: 2px 10px; border-radius: 4px; font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.4px; }
.arrival-meta { display: flex; flex-direction: column; gap: 4px; margin-top: 4px; }
.arrival-location { color: #8b7355; font-size: 12px; }
.arrival-stock { font-size: 12px; font-weight: 700; }
.in-stock { color: #16a34a; }
.out-stock { color: #dc2626; }
.arrival-colris-btn { display: inline-block; margin-top: 8px; background: #1a0f0a; color: #fdf6e3; text-decoration: none; padding: 9px 16px; border-radius: 6px; font-size: 12px; font-weight: 700; text-align: center; text-transform: uppercase; letter-spacing: 0.4px; transition: all 0.25s; }
.arrival-colris-btn:hover { background: #2c1810; transform: translateY(-1px); }

/* ── CHAT MESSAGES ── */
.chat-messages { flex: 1; overflow-y: auto; padding: 24px 20px; display: flex; flex-direction: column; gap: 16px; background: #f5f0e8; }
.chat-messages::-webkit-scrollbar { width: 4px; }
.chat-messages::-webkit-scrollbar-thumb { background: #e2d8cc; border-radius: 4px; }

/* ── WELCOME BANNER ── */
.welcome-banner { text-align: center; padding: 32px 20px 16px; margin: 0 auto; display: flex; flex-direction: column; align-items: center; width: 100%; max-width: 800px; align-self: center; }
.welcome-icon { font-size: 52px; margin-bottom: 16px; animation: popIn 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275); }

.quick-chips { display: flex; flex-wrap: wrap; gap: 8px; justify-content: center; margin: 20px 0 16px; max-width: 600px; }
.chip { background: white; border: 1.5px solid #e8dcc8; color: #5c3d2e; padding: 8px 16px; border-radius: 20px; cursor: pointer; font-size: 13px; font-weight: 600; transition: all 0.2s; white-space: nowrap; }
.chip:hover { border-color: #c9a84c; background: #fdf6e3; transform: translateY(-1px); }
.arrivals-chip { background: #1a0f0a; border-color: #1a0f0a; color: #c9a84c; }
.arrivals-chip:hover { background: #2c1810; }
.capability-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; margin: 20px auto 16px; max-width: 720px; width: 100%; }
.capability-card { background: white; border: 1.5px solid #e8dcc8; border-radius: 14px; padding: 16px 12px; cursor: pointer; transition: all 0.25s ease; text-align: center; display: flex; flex-direction: column; align-items: center; gap: 6px; box-shadow: 0 2px 8px rgba(92,61,46,0.06); }
.capability-card:hover { border-color: #c9a84c; background: #fdf6e3; transform: translateY(-3px); box-shadow: 0 8px 20px rgba(92,61,46,0.12); }
.arrivals-card { background: #1a0f0a; border-color: #1a0f0a; }
.arrivals-card .cap-icon, .arrivals-card .cap-label, .arrivals-card .cap-hint { color: #c9a84c !important; }
.arrivals-card:hover { background: #2c1810; border-color: #c9a84c; }
.cap-icon { font-size: 24px; }
.cap-label { font-size: 13px; font-weight: 700; color: #1a0f0a; }
.cap-hint { font-size: 11px; color: #8b7355; line-height: 1.3; }
.welcome-tip { color: #8b7355; font-size: 12px; margin-top: 8px; font-style: italic; }
@keyframes popIn { from { transform: scale(0) rotate(-10deg); opacity: 0; } to { transform: scale(1) rotate(0); opacity: 1; } }
.welcome-banner h3 { color: #1a0f0a; font-size: 22px; margin-bottom: 8px; font-weight: 800; letter-spacing: -0.3px; }
.welcome-sub { color: #8b7355; font-size: 14px; margin-bottom: 28px; }
.quick-actions { display: flex; flex-wrap: wrap; gap: 10px; justify-content: center; }
.quick-actions button { background: white; border: 1.5px solid #e2d8cc; color: #2c1810; padding: 9px 18px; border-radius: 8px; cursor: pointer; font-size: 13px; font-weight: 600; transition: all 0.25s; box-shadow: 0 1px 4px rgba(0,0,0,0.05); }
.quick-actions button:hover { border-color: #c9a84c; color: #1a0f0a; background: #fdf6e3; transform: translateY(-2px); box-shadow: 0 4px 12px rgba(0,0,0,0.08); }
.arrivals-quick-btn { background: #1a0f0a !important; color: #c9a84c !important; border-color: #1a0f0a !important; }
.arrivals-quick-btn:hover { background: #2c1810 !important; }

/* ── MESSAGES ── */
.message { display: flex; align-items: flex-end; gap: 10px; animation: msgIn 0.3s ease; }
@keyframes msgIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
.message.user { flex-direction: row-reverse; max-width: calc(100% - 20px); margin-left: auto; }
.message-avatar { font-size: 14px; width: 32px; height: 32px; border-radius: 8px; background: #1a0f0a; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.user-avatar { background: linear-gradient(135deg, #c9a84c, #b8960a); }
.message-bubble { max-width: 85%; padding: 12px 16px; border-radius: 16px; font-size: 14px; line-height: 1.7; }
.message.user .message-bubble { background: #1a0f0a; color: #fdf6e3; border-bottom-right-radius: 4px; box-shadow: 0 2px 8px rgba(0,0,0,0.15); }
.message.bot .message-bubble { background: white; color: #1a0f0a; border-bottom-left-radius: 4px; border: 1px solid rgba(0,0,0,0.07); box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
.message-bubble :deep(a) { color: #c9a84c; text-decoration: underline; word-break: break-all; }
.message-bubble :deep(strong) { color: #1a0f0a; font-weight: 700; }
.message-bubble :deep(.list-item) { display: flex; gap: 10px; margin: 8px 0; padding: 8px 12px; background: #fdf6e3; border-radius: 8px; border-left: 3px solid #c9a84c; }
.message-bubble :deep(.list-number) { background: #1a0f0a; color: #c9a84c; width: 22px; height: 22px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 11px; font-weight: 800; flex-shrink: 0; }
.message-bubble :deep(.bullet-item) { margin: 6px 0; padding-left: 8px; color: #1a0f0a; }

.chat-input-area { display: flex; flex-direction: column; background: white; border-top: 1px solid #e2d8cc; box-shadow: 0 -4px 20px rgba(0,0,0,0.06); }
.feature-toolbar { display: flex; gap: 8px; padding: 10px 16px 0; overflow-x: auto; }
.feature-toolbar::-webkit-scrollbar { display: none; }
.toolbar-btn { display: flex; align-items: center; gap: 6px; background: #faf7f2; border: 1.5px solid #e8dcc8; color: #5c3d2e; padding: 6px 14px; border-radius: 20px; cursor: pointer; font-size: 12px; font-weight: 600; white-space: nowrap; transition: all 0.2s; }
.toolbar-btn:hover { background: #fdf6e3; border-color: #c9a84c; color: #1a0f0a; }
.arrivals-toolbar-btn { background: #1a0f0a; border-color: #1a0f0a; color: #c9a84c; }
.arrivals-toolbar-btn:hover { background: #2c1810; border-color: #c9a84c; color: #c9a84c; }
.chat-input { display: flex; padding: 10px 16px 14px; gap: 10px; align-items: center; }
.modal-overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); z-index: 999; display: flex; align-items: center; justify-content: center; }
.feature-modal { background: white; border-radius: 20px; padding: 28px; width: 90%; max-width: 480px; max-height: 80vh; overflow-y: auto; box-shadow: 0 24px 80px rgba(0,0,0,0.3); z-index: 1000; position: relative; }
.modal-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
.modal-header h3 { color: #1a0f0a; font-size: 18px; font-weight: 800; margin: 0; }
.modal-close { background: none; border: none; cursor: pointer; color: #8b7355; font-size: 18px; }
.modal-sub { color: #8b7355; font-size: 13px; margin: 0 0 20px; }
.modal-form { display: flex; flex-direction: column; gap: 14px; }
.form-group { display: flex; flex-direction: column; gap: 6px; }
.form-group label { color: #5c3d2e; font-size: 12px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.5px; }
.form-group input, .form-group textarea, .form-group select { padding: 10px 14px; border: 1.5px solid #e2d8cc; border-radius: 8px; font-size: 14px; color: #1a0f0a; background: #faf7f2; outline: none; font-family: inherit; transition: all 0.2s; }
.form-group input:focus, .form-group textarea:focus, .form-group select:focus { border-color: #c9a84c; background: white; box-shadow: 0 0 0 3px rgba(201,168,76,0.12); }
.modal-submit-btn { background: #1a0f0a; color: #fdf6e3; border: none; padding: 13px; border-radius: 10px; font-size: 14px; font-weight: 700; cursor: pointer; transition: all 0.25s; }
.modal-submit-btn:hover { background: #2c1810; transform: translateY(-1px); }
.modal-submit-btn:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }
.req-msg { padding: 10px 14px; border-radius: 8px; font-size: 13px; font-weight: 600; }
.req-msg.success { background: #f0fdf4; border: 1px solid #bbf7d0; color: #16a34a; }
.req-msg.error { background: #fef2f2; border: 1px solid #fecaca; color: #dc2626; }
.citation-result { background: #fdf6e3; border: 1.5px solid #c9a84c; border-radius: 10px; padding: 14px; }
.cit-label { color: #8b5e3c; font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.5px; margin: 0 0 6px; }
.cit-text { color: #1a0f0a; font-size: 14px; line-height: 1.6; margin: 0 0 10px; font-style: italic; }
.copy-cit-btn { background: #1a0f0a; color: #fdf6e3; border: none; padding: 6px 14px; border-radius: 6px; font-size: 12px; font-weight: 700; cursor: pointer; }
.empty-requests { text-align: center; padding: 40px; color: #b8a898; font-size: 14px; }
.req-item { padding: 14px 0; border-bottom: 1px solid #f0e8dc; }
.req-item:last-child { border-bottom: none; }
.req-title { color: #1a0f0a; font-size: 15px; font-weight: 700; }
.req-author { color: #8b7355; font-size: 13px; margin: 2px 0; }
.req-status-row { display: flex; align-items: center; gap: 10px; margin-top: 6px; }
.req-badge { padding: 3px 10px; border-radius: 4px; font-size: 11px; font-weight: 700; text-transform: uppercase; }
.req-badge.pending { background: #fef9e7; color: #92400e; border: 1px solid #fde68a; }
.req-badge.approved { background: #f0fdf4; color: #16a34a; border: 1px solid #bbf7d0; }
.req-badge.rejected { background: #fef2f2; color: #dc2626; border: 1px solid #fecaca; }
.req-badge.ordered { background: #fdf6e3; color: #8b5e3c; border: 1px solid #e8dcc8; }
.req-date { font-size: 11px; color: #b8a898; }
.req-note { margin-top: 6px; font-size: 12px; color: #5c3d2e; background: #fdf6e3; padding: 6px 10px; border-radius: 6px; }
.file-preview-bar { padding: 8px 16px 0; }
.file-preview-card { display: flex; align-items: center; gap: 10px; background: #f0e8dc; border: 1px solid #e8dcc8; border-radius: 10px; padding: 8px 12px; width: fit-content; max-width: 300px; position: relative; }
.file-preview-img { width: 48px; height: 48px; object-fit: cover; border-radius: 6px; flex-shrink: 0; }
.file-preview-icon { width: 40px; height: 40px; background: #1a0f0a; border-radius: 8px; display: flex; align-items: center; justify-content: center; color: #c9a84c; flex-shrink: 0; }
.file-preview-info { display: flex; flex-direction: column; gap: 2px; overflow: hidden; }
.file-preview-name { color: #1a0f0a; font-size: 13px; font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; max-width: 180px; }
.file-preview-size { color: #8b7355; font-size: 11px; }
.file-preview-remove { background: rgba(0,0,0,0.1); border: none; cursor: pointer; color: #5c3d2e; width: 22px; height: 22px; border-radius: 50%; display: flex; align-items: center; justify-content: center; flex-shrink: 0; transition: all 0.2s; }
.file-preview-remove:hover { background: rgba(220,38,38,0.15); color: #dc2626; }
.attachment-card { display: flex; align-items: center; gap: 10px; background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2); border-radius: 10px; padding: 10px 14px; margin-bottom: 6px; max-width: 220px; }
.attachment-icon { width: 32px; height: 32px; background: rgba(255,255,255,0.15); border-radius: 6px; display: flex; align-items: center; justify-content: center; color: #fdf6e3; flex-shrink: 0; }
.attachment-info { display: flex; flex-direction: column; gap: 2px; overflow: hidden; }
.attachment-name { color: #fdf6e3; font-size: 13px; font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; max-width: 160px; }
.attachment-type { color: rgba(253,246,227,0.6); font-size: 11px; }
.message-actions { display: flex; gap: 6px; margin-top: 4px; }
.action-btn { background: transparent; border: 1px solid #e0d8cc; color: #a09080; width: 26px; height: 26px; border-radius: 6px; cursor: pointer; font-size: 12px; display: flex; align-items: center; justify-content: center; transition: all 0.2s; padding: 0; }
.action-btn:hover { border-color: #c9a84c; background: #fdf6e3; color: #5c3d2e; }
.active-up { background: #f0fdf4; border-color: #16a34a; }
.active-down { background: #fef2f2; border-color: #dc2626; }
.save-toggle-btn { background: rgba(201,168,76,0.15); border: 1px solid rgba(201,168,76,0.35); color: #c9a84c; padding: 6px 10px; border-radius: 6px; cursor: pointer; font-size: 14px; transition: all 0.2s; }
.save-toggle-btn:hover { background: #c9a84c; color: #1a0f0a; }
.saved-panel { position: absolute; top: 58px; right: 0; width: 340px; max-height: 480px; background: white; border: 1.5px solid #e8dcc8; border-radius: 0 0 16px 16px; box-shadow: 0 8px 32px rgba(0,0,0,0.12); z-index: 20; overflow-y: auto; }
.saved-header { display: flex; justify-content: space-between; align-items: center; padding: 14px 16px; border-bottom: 1px solid #f0e8dc; font-weight: 700; color: #1a0f0a; font-size: 14px; }
.close-saved { background: none; border: none; cursor: pointer; color: #8b7355; font-size: 16px; }
.saved-empty { padding: 24px; text-align: center; color: #b8a898; font-size: 13px; }
.saved-item { padding: 14px 16px; border-bottom: 1px solid #f5ede0; }
.saved-text { font-size: 13px; color: #1a0f0a; line-height: 1.5; margin: 0 0 8px; max-height: 80px; overflow: hidden; }
.saved-meta { display: flex; justify-content: space-between; align-items: center; }
.saved-meta span { font-size: 11px; color: #b8a898; }
.delete-saved { background: none; border: none; color: #dc2626; font-size: 11px; cursor: pointer; font-weight: 600; }
.did-you-know { background: #fdf6e3; border: 1px solid #e8dcc8; border-radius: 10px; padding: 10px 16px; margin-top: 8px; display: flex; align-items: center; gap: 10px; font-size: 13px; max-width: 720px; width: 100%; }
.dyk-label { font-weight: 700; color: #8b5e3c; white-space: nowrap; }
.dyk-text { color: #5c3d2e; }
.chat-header { position: relative; }
.handoff-card { background: white; border: 1.5px solid #c9a84c; border-radius: 14px; padding: 16px; box-shadow: 0 4px 16px rgba(201,168,76,0.15); animation: msgIn 0.3s ease; }
.handoff-title { color: #1a0f0a; font-size: 14px; font-weight: 700; margin: 0 0 4px; }
.handoff-sub { color: #8b7355; font-size: 13px; margin: 0 0 14px; }
.handoff-actions { margin-bottom: 14px; }
.handoff-btn.primary { display: inline-block; background: #1a0f0a; color: #fdf6e3; text-decoration: none; padding: 10px 18px; border-radius: 8px; font-size: 13px; font-weight: 700; transition: all 0.25s; }
.handoff-btn.primary:hover { background: #2c1810; transform: translateY(-1px); box-shadow: 0 4px 12px rgba(0,0,0,0.2); }
.suggestions-label { color: #8b7355; font-size: 12px; font-weight: 600; margin: 0 0 8px; text-transform: uppercase; letter-spacing: 0.5px; }
.suggestion-chips { display: flex; flex-wrap: wrap; gap: 8px; }
.chip { background: #fdf6e3; border: 1.5px solid #e8dcc8; color: #2c1810; padding: 6px 12px; border-radius: 20px; font-size: 12px; font-weight: 600; cursor: pointer; transition: all 0.2s; }
.chip:hover { background: #c9a84c; color: white; border-color: #c9a84c; transform: translateY(-1px); }
.typing-indicator { display: flex; align-items: center; gap: 4px; padding: 14px 18px !important; }
.typing-indicator span { width: 8px; height: 8px; background: #c9a84c; border-radius: 50%; animation: bounce 1.2s infinite ease-in-out; }
.typing-indicator span:nth-child(2) { animation-delay: 0.2s; }
.typing-indicator span:nth-child(3) { animation-delay: 0.4s; }
@keyframes bounce { 0%, 60%, 100% { transform: translateY(0); } 30% { transform: translateY(-8px); } }

/* ── INPUT BAR ── */
.attach-btn { width: 36px; height: 36px; background: #faf7f2; border: 1.5px solid #e8dcc8; border-radius: 8px; cursor: pointer; display: flex; align-items: center; justify-content: center; color: #8b7355; transition: all 0.2s; flex-shrink: 0; }
.attach-btn:hover { background: #fdf6e3; border-color: #c9a84c; color: #5c3d2e; }
.attached-file { display: flex; align-items: center; gap: 6px; background: #fdf6e3; border: 1px solid #c9a84c; border-radius: 6px; padding: 4px 8px; font-size: 12px; color: #5c3d2e; white-space: nowrap; max-width: 150px; overflow: hidden; }
.attached-file span { overflow: hidden; text-overflow: ellipsis; }
.remove-file { background: none; border: none; cursor: pointer; color: #8b7355; font-size: 14px; padding: 0; line-height: 1; flex-shrink: 0; }
.chat-input-old { display: none; }
.chat-input input { flex: 1; padding: 12px 18px; border: 1.5px solid #e2d8cc; border-radius: 10px; font-size: 14px; outline: none; background: #f5f0e8; color: #1a0f0a; font-family: 'Segoe UI', Arial, sans-serif; transition: all 0.25s; }
.chat-input input:focus { border-color: #c9a84c; background: white; box-shadow: 0 0 0 3px rgba(201,168,76,0.12); }
.chat-input input::placeholder { color: #b8a898; }
.send-btn { width: 44px; height: 44px; background: #1a0f0a; color: white; border: none; border-radius: 10px; font-size: 16px; cursor: pointer; display: flex; align-items: center; justify-content: center; transition: all 0.25s; flex-shrink: 0; box-shadow: 0 2px 8px rgba(0,0,0,0.2); }
.send-btn:hover { background: #2c1810; transform: scale(1.08); box-shadow: 0 4px 16px rgba(0,0,0,0.3); }
.send-btn:disabled { opacity: 0.4; cursor: not-allowed; transform: none; }
.mic-btn { width: 44px; height: 44px; background: #f5f0e8; border: 1.5px solid #e2d8cc; border-radius: 10px; font-size: 17px; cursor: pointer; display: flex; align-items: center; justify-content: center; flex-shrink: 0; transition: all 0.25s; }
.mic-btn:hover { background: #fdf6e3; border-color: #c9a84c; transform: scale(1.05); }
.mic-btn.recording { background: #dc2626; border-color: #dc2626; animation: micPulse 1s infinite; }
.arrivals-btn { width: 44px; height: 44px; background: #fdf6e3; border: 1.5px solid #c9a84c; border-radius: 10px; font-size: 17px; cursor: pointer; display: flex; align-items: center; justify-content: center; flex-shrink: 0; transition: all 0.25s; }
.arrivals-btn:hover { background: #c9a84c; transform: scale(1.05); }
@keyframes micPulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.1); } }
</style>