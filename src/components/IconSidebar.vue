<template>
  <div :class="['icon-sidebar', collapsed ? 'collapsed' : '']">
    <!-- Header -->
    <div class="sidebar-head">
      <div class="sidebar-logo" v-if="!collapsed">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="#c9a84c" stroke-width="2"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"/><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"/></svg>
        <span>COLRIS</span>
      </div>
      <button @click="collapsed = !collapsed" class="collapse-btn" :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'">
        <svg v-if="collapsed" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="3" y1="12" x2="21" y2="12"/><line x1="3" y1="6" x2="21" y2="6"/><line x1="3" y1="18" x2="21" y2="18"/></svg>
        <svg v-else width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="3" y1="12" x2="21" y2="12"/><line x1="3" y1="6" x2="21" y2="6"/><line x1="3" y1="18" x2="21" y2="18"/></svg>
      </button>
    </div>

    <!-- Nav Items -->
    <div class="sidebar-nav">

      <button class="nav-btn" @click="$emit('new-chat')">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>
        <span>New Chat</span>
      </button>

      <button :class="['nav-btn', activePanel === 'conversations' ? 'active' : '']" @click="toggle('conversations'); loadConversations()">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/></svg>
        <span>Conversations</span>
      </button>

      <button :class="['nav-btn', activePanel === 'search' ? 'active' : '']" @click="toggle('search')">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
        <span>Search Books</span>
      </button>

      <button :class="['nav-btn', activePanel === 'links' ? 'active' : '']" @click="toggle('links')">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/></svg>
        <span>Quick Links</span>
      </button>

      <div class="nav-divider" v-if="!isAdmin"><span>Library Tools</span></div>

      <template v-if="!isAdmin">
        <button :class="['nav-btn', activePanel === 'request' ? 'active' : '']" @click="toggle('request')">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
          <span>Request Book</span>
        </button>

        <button :class="['nav-btn', activePanel === 'citation' ? 'active' : '']" @click="toggle('citation')">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>
          <span>Citation Generator</span>
        </button>

        <button :class="['nav-btn', activePanel === 'myrequests' ? 'active' : '']" @click="toggle('myrequests'); loadMyRequests()">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M9 11l3 3L22 4"/><path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"/></svg>
          <span>My Requests</span>
          <span v-if="pendingCount > 0" class="nav-badge">{{ pendingCount }}</span>
        </button>

        <button :class="['nav-btn', activePanel === 'saved' ? 'active' : '']" @click="toggle('saved')">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z"/></svg>
          <span>Saved Answers</span>
          <span v-if="savedAnswers.length > 0" class="nav-badge">{{ savedAnswers.length }}</span>
        </button>

        <button :class="['nav-btn', activePanel === 'stats' ? 'active' : '']" @click="toggle('stats')">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/></svg>
          <span>My Activity</span>
        </button>

        <button :class="['nav-btn', activePanel === 'arrivals' ? 'active' : '']" @click="toggle('arrivals'); $emit('open-arrivals')">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"/><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"/><line x1="12" y1="6" x2="12" y2="12"/><line x1="9" y1="9" x2="15" y2="9"/></svg>
          <span>New Arrivals</span>
        </button>
      </template>

      <template v-if="isAdmin">
        <div class="nav-divider"><span>Admin</span></div>
        <button class="nav-btn admin-btn" @click="$emit('open-admin')">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></svg>
          <span>Admin Panel</span>
        </button>
      </template>
    </div>

    <!-- Panel -->
    <transition name="panel-slide">
      <div v-if="activePanel" class="slide-panel">
        <div class="panel-header">
          <h3>{{ panelTitle }}</h3>
          <button @click="activePanel = null" class="panel-close">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
          </button>
        </div>

        <div v-if="activePanel === 'conversations'" class="panel-content">
          <button @click="$emit('new-chat'); activePanel = null" class="new-chat-btn">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>
            New Conversation
          </button>
          <div v-if="loadingConvs" class="panel-loading">Loading...</div>
          <div v-else-if="conversations.length === 0" class="panel-empty">No conversations yet.</div>
          <div v-for="conv in conversations" :key="conv.session_id" class="conv-item" @click="$emit('load-session', conv.session_id); activePanel = null">
            <div class="conv-title">{{ conv.first_message || 'New Conversation' }}</div>
            <div class="conv-date">{{ formatDate(conv.started_at) }}</div>
          </div>
        </div>

        <div v-if="activePanel === 'search'" class="panel-content">
          <div class="panel-search-bar">
            <input v-model="searchQuery" @keyup.enter="searchBooks" placeholder="Search by title, author or subject..." />
            <button @click="searchBooks" class="panel-search-btn">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
            </button>
          </div>
          <div v-if="isSearching" class="panel-loading">Searching...</div>
          <div v-else-if="searchResults.length === 0 && hasSearched" class="panel-empty">No results for "{{ searchQuery }}"</div>
          <div v-for="book in searchResults" :key="book.link" class="panel-book-item">
            <div class="panel-book-title">{{ book.title }}</div>
            <div class="panel-book-author">{{ book.author }}</div>
            <a :href="book.link" target="_blank" class="panel-book-link">View in COLRIS →</a>
          </div>
        </div>

        <div v-if="activePanel === 'request'" class="panel-content">
          <p class="panel-desc">Submit a book request to the librarian.</p>
          <div class="panel-form">
            <input v-model="reqTitle" placeholder="Book Title *" class="panel-input" />
            <input v-model="reqAuthor" placeholder="Author (optional)" class="panel-input" />
            <textarea v-model="reqReason" placeholder="Why do you need this book?" class="panel-textarea" rows="3"></textarea>
            <div v-if="reqMsg" :class="['panel-msg', reqSuccess ? 'success' : 'error']">{{ reqMsg }}</div>
            <button @click="submitRequest" class="panel-submit-btn" :disabled="isSubmitting">{{ isSubmitting ? 'Submitting...' : 'Submit Request' }}</button>
          </div>
        </div>

        <div v-if="activePanel === 'citation'" class="panel-content">
          <p class="panel-desc">Generate a formatted academic citation.</p>
          <div class="panel-form">
            <input v-model="citAuthor" placeholder="Author(s)" class="panel-input" />
            <input v-model="citTitle" placeholder="Title of Book/Article" class="panel-input" />
            <input v-model="citYear" placeholder="Year of Publication" class="panel-input" />
            <input v-model="citPublisher" placeholder="Publisher" class="panel-input" />
            <input v-model="citCity" placeholder="City (optional)" class="panel-input" />
            <select v-model="citStyle" class="panel-input">
              <option value="APA">APA</option>
              <option value="MLA">MLA</option>
              <option value="Harvard">Harvard</option>
            </select>
            <button @click="generateCit" class="panel-submit-btn">Generate Citation</button>
            <div v-if="citResult" class="cit-result-box">
              <p class="cit-result-text">{{ citResult }}</p>
              <button @click="copyCit" class="copy-cit">{{ citCopied ? 'Copied!' : 'Copy' }}</button>
            </div>
          </div>
        </div>

        <div v-if="activePanel === 'myrequests'" class="panel-content">
          <div v-if="loadingReqs" class="panel-loading">Loading...</div>
          <div v-else-if="myRequests.length === 0" class="panel-empty">No book requests yet.</div>
          <div v-for="req in myRequests" :key="req.id" class="req-card">
            <div class="req-card-title">{{ req.title }}</div>
            <div v-if="req.author" class="req-card-author">by {{ req.author }}</div>
            <div class="req-card-bottom">
              <span :class="['req-status', req.status]">{{ req.status }}</span>
              <span class="req-date">{{ new Date(req.created_at).toLocaleDateString() }}</span>
            </div>
            <div v-if="req.admin_note" class="req-note">📌 {{ req.admin_note }}</div>
          </div>
        </div>

        <div v-if="activePanel === 'saved'" class="panel-content">
          <div v-if="savedAnswers.length === 0" class="panel-empty">No saved answers yet.</div>
          <div v-for="(ans, i) in savedAnswers" :key="i" class="saved-card">
            <p class="saved-card-text" v-html="formatMessage(ans.text)"></p>
            <div class="saved-card-bottom">
              <span class="saved-time">{{ ans.time }}</span>
              <button @click="savedAnswers.splice(i, 1)" class="remove-saved">Remove</button>
            </div>
          </div>
        </div>

        <div v-if="activePanel === 'links'" class="panel-content">
          <p class="panel-desc">One-tap access to academic databases.</p>
          <div class="links-grid">
            <a v-for="link in quickLinks" :key="link.name" :href="link.url" target="_blank" class="link-card">
              <div class="link-icon">{{ link.icon }}</div>
              <div class="link-name">{{ link.name }}</div>
              <div class="link-desc">{{ link.desc }}</div>
            </a>
          </div>
        </div>

        <div v-if="activePanel === 'stats'" class="panel-content">
          <p class="panel-desc">Your activity this session.</p>
          <div class="stats-grid">
            <div class="stat-card"><div class="stat-card-num">{{ stats.questions }}</div><div class="stat-card-label">Questions</div></div>
            <div class="stat-card"><div class="stat-card-num">{{ stats.requests }}</div><div class="stat-card-label">Requests</div></div>
            <div class="stat-card"><div class="stat-card-num">{{ stats.citations }}</div><div class="stat-card-label">Citations</div></div>
            <div class="stat-card"><div class="stat-card-num">{{ stats.saved }}</div><div class="stat-card-label">Saved</div></div>
          </div>
        </div>
      </div>
    </transition>

    <!-- Bottom -->
    <div class="sidebar-bottom">
      <button class="nav-btn signout-btn" @click="$emit('logout')">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>
        <span>Sign Out</span>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "IconSidebar",
  props: {
    savedAnswers: { type: Array, default: () => [] },
    stats: { type: Object, default: () => ({ questions: 0, requests: 0, citations: 0, saved: 0 }) },
    token: { type: String, default: "" },
    formatMessage: { type: Function, default: (t) => t },
    isAdmin: { type: Boolean, default: false }
  },
  emits: ["new-chat", "open-arrivals", "logout", "open-admin", "load-session"],
  data() {
    return {
      collapsed: true,
      activePanel: null,
      searchQuery: "", searchResults: [], isSearching: false, hasSearched: false,
      reqTitle: "", reqAuthor: "", reqReason: "", reqMsg: "", reqSuccess: false, isSubmitting: false,
      citAuthor: "", citTitle: "", citYear: "", citPublisher: "", citCity: "", citStyle: "APA", citResult: "", citCopied: false,
      myRequests: [], loadingReqs: false, pendingCount: 0,
      conversations: [], loadingConvs: false,
      quickLinks: [
        { icon: "📚", name: "COLRIS", desc: "CU Library Catalogue", url: "https://colris.covenantuniversity.edu.ng" },
        { icon: "🔬", name: "JSTOR", desc: "Academic Journals", url: "https://www.jstor.org" },
        { icon: "💡", name: "IEEE Xplore", desc: "Tech & Engineering", url: "https://ieeexplore.ieee.org" },
        { icon: "🌍", name: "Google Scholar", desc: "Academic Search", url: "https://scholar.google.com" },
        { icon: "🧬", name: "PubMed", desc: "Medical Research", url: "https://pubmed.ncbi.nlm.nih.gov" },
        { icon: "📖", name: "ResearchGate", desc: "Research Network", url: "https://www.researchgate.net" },
        { icon: "🏛️", name: "Academia.edu", desc: "Academic Papers", url: "https://www.academia.edu" },
        { icon: "📰", name: "ScienceDirect", desc: "Scientific Articles", url: "https://www.sciencedirect.com" }
      ]
    }
  },
  computed: {
    panelTitle() {
      const t = { conversations: "Conversations", search: "Search Books", request: "Request a Book", citation: "Citation Generator", myrequests: "My Requests", saved: "Saved Answers", links: "Quick Links", stats: "My Activity", arrivals: "New Arrivals" }
      return t[this.activePanel] || ""
    }
  },
  methods: {
    toggle(panel) { this.activePanel = this.activePanel === panel ? null : panel },
    formatDate(dateStr) {
      const d = new Date(dateStr), now = new Date(), diff = now - d
      if (diff < 86400000) return d.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })
      if (diff < 604800000) return d.toLocaleDateString([], { weekday: "short" })
      return d.toLocaleDateString([], { month: "short", day: "numeric" })
    },
    async loadConversations() {
      this.loadingConvs = true
      try {
        const res = await fetch("https://colris-chatbot-backend-production.up.railway.app/api/chat/history", {
          headers: { "Authorization": "Bearer " + this.token, "Accept": "application/json" }
        })
        const data = await res.json()
        this.conversations = (data.history || []).map(c => ({ ...c, first_message: c.first_message ? c.first_message.substring(0, 40) : "New Conversation" }))
      } catch (e) { this.conversations = [] }
      this.loadingConvs = false
    },
    async searchBooks() {
      if (!this.searchQuery.trim()) return
      this.isSearching = true; this.hasSearched = true
      try {
        const res = await fetch("https://openlibrary.org/search.json?q=" + encodeURIComponent(this.searchQuery) + "&limit=5")
        const data = await res.json()
        this.searchResults = (data.docs || []).slice(0, 5).map(b => ({
          title: b.title, author: b.author_name ? b.author_name[0] : "Unknown",
          link: "https://colris.covenantuniversity.edu.ng/discovery/search?query=any,contains," + encodeURIComponent(b.title) + "&tab=Everything&search_scope=MyInst_and_CI&vid=234COU_INST:VU1&lang=en&offset=0"
        }))
      } catch (e) { this.searchResults = [] }
      this.isSearching = false
    },
    async submitRequest() {
      if (!this.reqTitle) { this.reqMsg = "Please enter the book title."; this.reqSuccess = false; return }
      this.isSubmitting = true
      try {
        const res = await fetch("https://colris-chatbot-backend-production.up.railway.app/api/book-requests", {
          method: "POST",
          headers: { "Content-Type": "application/json", "Authorization": "Bearer " + this.token },
          body: JSON.stringify({ title: this.reqTitle, author: this.reqAuthor, reason: this.reqReason })
        })
        if (res.ok) { this.reqSuccess = true; this.reqMsg = "Request submitted!"; this.reqTitle = ""; this.reqAuthor = ""; this.reqReason = "" }
        else { this.reqMsg = "Failed. Try again."; this.reqSuccess = false }
      } catch (e) { this.reqMsg = "Network error."; this.reqSuccess = false }
      this.isSubmitting = false
    },
    generateCit() {
      if (!this.citTitle || !this.citAuthor || !this.citYear) { alert("Fill Author, Title and Year."); return }
      if (this.citStyle === "APA") this.citResult = this.citAuthor + " (" + this.citYear + "). " + this.citTitle + ". " + (this.citCity ? this.citCity + ": " : "") + this.citPublisher + "."
      else if (this.citStyle === "MLA") this.citResult = this.citAuthor + ". " + this.citTitle + ". " + this.citPublisher + ", " + this.citYear + "."
      else this.citResult = this.citAuthor + " " + this.citYear + ", " + this.citTitle + ", " + this.citPublisher + (this.citCity ? ", " + this.citCity : "") + "."
    },
    copyCit() { navigator.clipboard.writeText(this.citResult).then(() => { this.citCopied = true; setTimeout(() => this.citCopied = false, 2000) }) },
    async loadMyRequests() {
      this.loadingReqs = true
      try {
        const res = await fetch("https://colris-chatbot-backend-production.up.railway.app/api/book-requests/my", {
          headers: { "Authorization": "Bearer " + this.token, "Accept": "application/json" }
        })
        const data = await res.json()
        this.myRequests = data.requests || []
        this.pendingCount = this.myRequests.filter(r => r.status === "pending").length
      } catch (e) { this.myRequests = [] }
      this.loadingReqs = false
    }
  }
}
</script>

<style scoped>
.icon-sidebar { width: 220px; min-width: 220px; height: 100vh; background: #130d09; border-right: 1px solid rgba(201,168,76,0.1); display: flex; flex-direction: column; flex-shrink: 0; position: relative; overflow: visible; transition: width 0.25s ease, min-width 0.25s ease; }
.icon-sidebar.collapsed { width: 56px; min-width: 56px; }

.sidebar-logo { display: flex; align-items: center; gap: 8px; color: #c9a84c; font-size: 14px; font-weight: 800; letter-spacing: 1px; }
.sidebar-nav { flex: 1; overflow-y: auto; padding: 8px 8px; display: flex; flex-direction: column; gap: 2px; }
.sidebar-nav::-webkit-scrollbar { width: 3px; }
.sidebar-nav::-webkit-scrollbar-thumb { background: rgba(201,168,76,0.2); border-radius: 2px; }
.nav-btn { display: flex; align-items: center; gap: 10px; width: 100%; padding: 8px 10px; border-radius: 8px; background: none; border: none; color: #8b7355; cursor: pointer; font-size: 13px; font-weight: 500; text-align: left; transition: all 0.2s; position: relative; }
.nav-btn:hover { background: rgba(201,168,76,0.08); color: #fdf6e3; }
.nav-btn.active { background: rgba(201,168,76,0.12); color: #c9a84c; }
.nav-btn svg { flex-shrink: 0; }
.admin-btn { color: #c9a84c; }
.admin-btn:hover { background: rgba(201,168,76,0.12); color: #e0c060; }
.signout-btn:hover { background: rgba(220,38,38,0.1); color: #ef4444; }
.nav-badge { margin-left: auto; background: #c9a84c; color: #0f0905; width: 18px; height: 18px; border-radius: 50%; font-size: 10px; font-weight: 800; display: flex; align-items: center; justify-content: center; }
.nav-divider { padding: 12px 10px 4px; font-size: 10px; font-weight: 700; color: #4a3728; text-transform: uppercase; letter-spacing: 1px; }
.sidebar-bottom { padding: 8px; border-top: 1px solid rgba(201,168,76,0.08); }

/* Slide Panel */
.slide-panel { position: absolute; left: 220px; top: 0; width: 280px; height: 100vh; background: #1a0f0a; border-right: 1px solid rgba(201,168,76,0.12); display: flex; flex-direction: column; z-index: 50; box-shadow: 4px 0 20px rgba(0,0,0,0.3); }
.panel-header { display: flex; justify-content: space-between; align-items: center; padding: 16px; border-bottom: 1px solid rgba(201,168,76,0.1); flex-shrink: 0; }
.panel-header h3 { color: #fdf6e3; font-size: 14px; font-weight: 700; }
.panel-close { background: none; border: none; cursor: pointer; color: #6b5a4e; transition: color 0.2s; padding: 4px; }
.panel-close:hover { color: #c9a84c; }
.panel-content { flex: 1; overflow-y: auto; padding: 14px; display: flex; flex-direction: column; gap: 10px; }
.panel-desc { color: #8b7355; font-size: 12px; line-height: 1.5; }
.panel-empty { color: #6b5a4e; font-size: 13px; text-align: center; padding: 32px 0; }
.panel-loading { color: #8b7355; font-size: 13px; text-align: center; padding: 20px 0; }
.new-chat-btn { display: flex; align-items: center; gap: 8px; background: rgba(201,168,76,0.12); border: 1px solid rgba(201,168,76,0.25); color: #c9a84c; padding: 9px 12px; border-radius: 8px; cursor: pointer; font-size: 13px; font-weight: 600; width: 100%; transition: all 0.2s; }
.new-chat-btn:hover { background: rgba(201,168,76,0.2); }
.conv-item { padding: 9px 10px; border-radius: 7px; cursor: pointer; transition: all 0.2s; }
.conv-item:hover { background: rgba(255,255,255,0.05); }
.conv-title { color: #c4aa94; font-size: 13px; font-weight: 500; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; margin-bottom: 2px; }
.conv-date { color: #4a3728; font-size: 11px; }
.panel-search-bar { display: flex; gap: 8px; }
.panel-search-bar input { flex: 1; background: rgba(255,255,255,0.05); border: 1px solid rgba(201,168,76,0.2); color: #fdf6e3; padding: 8px 10px; border-radius: 7px; font-size: 12px; outline: none; }
.panel-search-bar input:focus { border-color: #c9a84c; }
.panel-search-btn { background: #c9a84c; border: none; color: #0f0905; width: 34px; border-radius: 7px; cursor: pointer; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.panel-book-item { background: rgba(255,255,255,0.03); border: 1px solid rgba(201,168,76,0.1); border-radius: 8px; padding: 10px; }
.panel-book-title { color: #fdf6e3; font-size: 12px; font-weight: 700; margin-bottom: 3px; }
.panel-book-author { color: #8b7355; font-size: 11px; margin-bottom: 6px; }
.panel-book-link { color: #c9a84c; font-size: 11px; font-weight: 600; text-decoration: none; }
.panel-form { display: flex; flex-direction: column; gap: 8px; }
.panel-input { background: rgba(255,255,255,0.05); border: 1px solid rgba(201,168,76,0.2); color: #fdf6e3; padding: 8px 10px; border-radius: 7px; font-size: 12px; outline: none; font-family: inherit; width: 100%; }
.panel-input:focus { border-color: #c9a84c; }
.panel-textarea { background: rgba(255,255,255,0.05); border: 1px solid rgba(201,168,76,0.2); color: #fdf6e3; padding: 8px 10px; border-radius: 7px; font-size: 12px; outline: none; font-family: inherit; width: 100%; resize: none; }
.panel-submit-btn { background: #c9a84c; color: #0f0905; border: none; padding: 9px; border-radius: 7px; font-size: 13px; font-weight: 700; cursor: pointer; }
.panel-submit-btn:hover { background: #e0c060; }
.panel-submit-btn:disabled { opacity: 0.6; cursor: not-allowed; }
.panel-msg { padding: 7px 10px; border-radius: 7px; font-size: 12px; font-weight: 600; }
.panel-msg.success { background: rgba(22,163,74,0.12); color: #4ade80; }
.panel-msg.error { background: rgba(220,38,38,0.12); color: #f87171; }
.cit-result-box { background: rgba(201,168,76,0.07); border: 1px solid rgba(201,168,76,0.2); border-radius: 7px; padding: 10px; }
.cit-result-text { color: #fdf6e3; font-size: 12px; line-height: 1.6; margin-bottom: 7px; font-style: italic; }
.copy-cit { background: #c9a84c; color: #0f0905; border: none; padding: 4px 10px; border-radius: 5px; font-size: 11px; font-weight: 700; cursor: pointer; }
.req-card { background: rgba(255,255,255,0.03); border: 1px solid rgba(201,168,76,0.1); border-radius: 8px; padding: 10px; }
.req-card-title { color: #fdf6e3; font-size: 12px; font-weight: 700; margin-bottom: 2px; }
.req-card-author { color: #8b7355; font-size: 11px; margin-bottom: 6px; }
.req-card-bottom { display: flex; justify-content: space-between; align-items: center; }
.req-status { padding: 2px 7px; border-radius: 4px; font-size: 10px; font-weight: 700; text-transform: uppercase; }
.req-status.pending { background: rgba(234,179,8,0.12); color: #fbbf24; }
.req-status.approved { background: rgba(22,163,74,0.12); color: #4ade80; }
.req-status.rejected { background: rgba(220,38,38,0.12); color: #f87171; }
.req-status.ordered { background: rgba(201,168,76,0.12); color: #c9a84c; }
.req-date { font-size: 10px; color: #4a3728; }
.req-note { margin-top: 6px; font-size: 11px; color: #c9a84c; background: rgba(201,168,76,0.07); padding: 5px 7px; border-radius: 5px; }
.saved-card { background: rgba(255,255,255,0.03); border: 1px solid rgba(201,168,76,0.1); border-radius: 8px; padding: 10px; }
.saved-card-text { color: #c4aa94; font-size: 11px; line-height: 1.5; max-height: 70px; overflow: hidden; margin-bottom: 7px; }
.saved-card-bottom { display: flex; justify-content: space-between; align-items: center; }
.saved-time { font-size: 10px; color: #4a3728; }
.remove-saved { background: none; border: none; color: #ef4444; font-size: 10px; font-weight: 600; cursor: pointer; }
.links-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 7px; }
.link-card { background: rgba(255,255,255,0.03); border: 1px solid rgba(201,168,76,0.1); border-radius: 8px; padding: 10px 8px; text-decoration: none; transition: all 0.2s; display: block; }
.link-card:hover { border-color: #c9a84c; background: rgba(201,168,76,0.07); }
.link-icon { font-size: 18px; margin-bottom: 4px; }
.link-name { color: #fdf6e3; font-size: 11px; font-weight: 700; margin-bottom: 2px; }
.link-desc { color: #4a3728; font-size: 10px; }
.stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 7px; }
.stat-card { background: rgba(255,255,255,0.03); border: 1px solid rgba(201,168,76,0.1); border-radius: 8px; padding: 14px 10px; text-align: center; }
.stat-card-num { font-size: 24px; font-weight: 900; color: #c9a84c; }
.stat-card-label { font-size: 10px; color: #4a3728; margin-top: 3px; }
.collapsed .nav-btn span { display: none; }
.collapsed .nav-divider { display: none; }
.collapsed .sidebar-logo { display: none; }
.collapsed .nav-badge { display: none; }
.collapse-btn { background: none; border: none; cursor: pointer; color: #6b5a4e; padding: 4px; display: flex; align-items: center; justify-content: center; transition: color 0.2s; margin-left: auto; }
.collapse-btn:hover { color: #c9a84c; }
.sidebar-head { display: flex; align-items: center; padding: 14px 10px 10px; border-bottom: 1px solid rgba(201,168,76,0.08); gap: 8px; }
.panel-slide-enter-active, .panel-slide-leave-active { transition: all 0.2s ease; }
.panel-slide-enter-from, .panel-slide-leave-to { transform: translateX(-10px); opacity: 0; }
</style>