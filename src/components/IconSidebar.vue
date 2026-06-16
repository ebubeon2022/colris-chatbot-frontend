<template>
  <div class="icon-sidebar">
    <div class="sidebar-icons">
      <div class="icon-group top">

        <div class="icon-item" @click="emit('new-chat')" title="New Chat">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="10" y1="10" x2="14" y2="10"/></svg>
          <span class="icon-tooltip">New Chat</span>
        </div>

        <div :class="['icon-item', activePanel === 'search' ? 'active' : '']" @click="toggle('search')" title="Search Books">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
          <span class="icon-tooltip">Search Books</span>
        </div>

        <div :class="['icon-item', activePanel === 'request' ? 'active' : '']" @click="toggle('request')" title="Request Book">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
          <span class="icon-tooltip">Request Book</span>
        </div>

        <div :class="['icon-item', activePanel === 'citation' ? 'active' : '']" @click="toggle('citation')" title="Citation Generator">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><polyline points="10 9 9 9 8 9"/></svg>
          <span class="icon-tooltip">Citation Generator</span>
        </div>

        <div :class="['icon-item', activePanel === 'myrequests' ? 'active' : '']" @click="toggle('myrequests'); loadMyRequests()" title="My Requests">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M9 11l3 3L22 4"/><path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"/></svg>
          <span class="icon-tooltip">My Requests</span>
          <span v-if="pendingCount > 0" class="badge">{{ pendingCount }}</span>
        </div>

        <div :class="['icon-item', activePanel === 'saved' ? 'active' : '']" @click="toggle('saved')" title="Saved Answers">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z"/></svg>
          <span class="icon-tooltip">Saved Answers</span>
          <span v-if="savedAnswers.length > 0" class="badge">{{ savedAnswers.length }}</span>
        </div>

        <div :class="['icon-item', activePanel === 'links' ? 'active' : '']" @click="toggle('links')" title="Quick Links">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/></svg>
          <span class="icon-tooltip">Quick Links</span>
        </div>

        <div :class="['icon-item', activePanel === 'stats' ? 'active' : '']" @click="toggle('stats')" title="My Activity">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/></svg>
          <span class="icon-tooltip">My Activity</span>
        </div>

        <div :class="['icon-item', activePanel === 'arrivals' ? 'active' : '']" @click="toggle('arrivals'); $emit('open-arrivals')" title="New Arrivals">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"/><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"/><line x1="12" y1="6" x2="12" y2="12"/><line x1="9" y1="9" x2="15" y2="9"/></svg>
          <span class="icon-tooltip">New Arrivals</span>
        </div>

      </div>

      <div class="icon-group bottom">
        <div class="icon-item logout" @click="$emit('logout')" title="Sign Out">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>
          <span class="icon-tooltip">Sign Out</span>
        </div>
      </div>
    </div>

    <!-- Slide panels -->
    <transition name="panel-slide">
      <div v-if="activePanel" class="slide-panel">
        <div class="panel-header">
          <h3>{{ panelTitle }}</h3>
          <button @click="activePanel = null" class="panel-close">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
          </button>
        </div>

        <!-- Search Panel -->
        <div v-if="activePanel === 'search'" class="panel-content">
          <div class="panel-search-bar">
            <input v-model="searchQuery" @keyup.enter="searchBooks" placeholder="Search by title, author or subject..." />
            <button @click="searchBooks" class="panel-search-btn">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
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

        <!-- Request Panel -->
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

        <!-- Citation Panel -->
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

        <!-- My Requests Panel -->
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

        <!-- Saved Panel -->
        <div v-if="activePanel === 'saved'" class="panel-content">
          <div v-if="savedAnswers.length === 0" class="panel-empty">No saved answers yet. Click the bookmark icon on any response.</div>
          <div v-for="(ans, i) in savedAnswers" :key="i" class="saved-card">
            <p class="saved-card-text" v-html="formatMessage(ans.text)"></p>
            <div class="saved-card-bottom">
              <span class="saved-time">{{ ans.time }}</span>
              <button @click="savedAnswers.splice(i, 1)" class="remove-saved">Remove</button>
            </div>
          </div>
        </div>

        <!-- Quick Links Panel -->
        <div v-if="activePanel === 'links'" class="panel-content">
          <p class="panel-desc">One-tap access to academic databases and resources.</p>
          <div class="links-grid">
            <a v-for="link in quickLinks" :key="link.name" :href="link.url" target="_blank" class="link-card">
              <div class="link-icon">{{ link.icon }}</div>
              <div class="link-name">{{ link.name }}</div>
              <div class="link-desc">{{ link.desc }}</div>
            </a>
          </div>
        </div>

        <!-- Stats Panel -->
        <div v-if="activePanel === 'stats'" class="panel-content">
          <p class="panel-desc">Your activity this session.</p>
          <div class="stats-grid">
            <div class="stat-card">
              <div class="stat-card-num">{{ stats.questions }}</div>
              <div class="stat-card-label">Questions Asked</div>
            </div>
            <div class="stat-card">
              <div class="stat-card-num">{{ stats.requests }}</div>
              <div class="stat-card-label">Books Requested</div>
            </div>
            <div class="stat-card">
              <div class="stat-card-num">{{ stats.citations }}</div>
              <div class="stat-card-label">Citations Generated</div>
            </div>
            <div class="stat-card">
              <div class="stat-card-num">{{ stats.saved }}</div>
              <div class="stat-card-label">Answers Saved</div>
            </div>
          </div>
        </div>

      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: "IconSidebar",
  props: {
    savedAnswers: { type: Array, default: () => [] },
    stats: { type: Object, default: () => ({ questions: 0, requests: 0, citations: 0, saved: 0 }) },
    token: { type: String, default: "" },
    formatMessage: { type: Function, default: (t) => t }
  },
  emits: ["new-chat", "open-arrivals", "logout", "search-books"],
  data() {
    return {
      activePanel: null,
      searchQuery: "",
      searchResults: [],
      isSearching: false,
      hasSearched: false,
      reqTitle: "",
      reqAuthor: "",
      reqReason: "",
      reqMsg: "",
      reqSuccess: false,
      isSubmitting: false,
      citAuthor: "",
      citTitle: "",
      citYear: "",
      citPublisher: "",
      citCity: "",
      citStyle: "APA",
      citResult: "",
      citCopied: false,
      myRequests: [],
      loadingReqs: false,
      pendingCount: 0,
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
      const titles = { search: "Search Books", request: "Request a Book", citation: "Citation Generator", myrequests: "My Requests", saved: "Saved Answers", links: "Quick Links", stats: "My Activity", arrivals: "New Arrivals" }
      return titles[this.activePanel] || ""
    }
  },
  methods: {
    toggle(panel) {
      this.activePanel = this.activePanel === panel ? null : panel
    },
    emit(event) {
      this.$emit(event)
    },
    async searchBooks() {
      if (!this.searchQuery.trim()) return
      this.isSearching = true
      this.hasSearched = true
      try {
        const res = await fetch("https://openlibrary.org/search.json?q=" + encodeURIComponent(this.searchQuery) + "&limit=5")
        const data = await res.json()
        this.searchResults = (data.docs || []).slice(0, 5).map(b => ({
          title: b.title,
          author: b.author_name ? b.author_name[0] : "Unknown",
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
        if (res.ok) {
          this.reqSuccess = true
          this.reqMsg = "Request submitted successfully!"
          this.reqTitle = ""; this.reqAuthor = ""; this.reqReason = ""
        } else { this.reqMsg = "Failed to submit. Please try again."; this.reqSuccess = false }
      } catch (e) { this.reqMsg = "Network error."; this.reqSuccess = false }
      this.isSubmitting = false
    },
    generateCit() {
      if (!this.citTitle || !this.citAuthor || !this.citYear) { alert("Please fill Author, Title and Year."); return }
      if (this.citStyle === "APA") this.citResult = this.citAuthor + " (" + this.citYear + "). " + this.citTitle + ". " + (this.citCity ? this.citCity + ": " : "") + this.citPublisher + "."
      else if (this.citStyle === 'MLA') this.citResult = this.citAuthor + '. "' + this.citTitle + '." ' + this.citPublisher + ', ' + this.citYear + '.'
      else this.citResult = this.citAuthor + " " + this.citYear + ", " + this.citTitle + ", " + this.citPublisher + (this.citCity ? ", " + this.citCity : "") + "."
    },
    copyCit() {
      navigator.clipboard.writeText(this.citResult).then(() => { this.citCopied = true; setTimeout(() => this.citCopied = false, 2000) })
    },
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
.icon-sidebar { display: flex; height: 100vh; position: relative; }
.sidebar-icons { width: 56px; background: #111; border-right: 1px solid rgba(201,168,76,0.12); display: flex; flex-direction: column; justify-content: space-between; padding: 12px 0; flex-shrink: 0; }
.icon-group { display: flex; flex-direction: column; gap: 4px; padding: 0 8px; }
.icon-item { position: relative; width: 40px; height: 40px; display: flex; align-items: center; justify-content: center; border-radius: 10px; cursor: pointer; color: #6b5a4e; transition: all 0.2s; }
.icon-item:hover { background: rgba(201,168,76,0.12); color: #c9a84c; }
.icon-item.active { background: rgba(201,168,76,0.18); color: #c9a84c; }
.icon-item.logout:hover { background: rgba(220,38,38,0.12); color: #ef4444; }
.icon-tooltip { position: absolute; left: 52px; background: #1a0f0a; color: #fdf6e3; padding: 5px 10px; border-radius: 6px; font-size: 12px; font-weight: 600; white-space: nowrap; opacity: 0; pointer-events: none; transition: opacity 0.2s; z-index: 100; border: 1px solid rgba(201,168,76,0.2); }
.icon-item:hover .icon-tooltip { opacity: 1; }
.badge { position: absolute; top: 4px; right: 4px; background: #c9a84c; color: #0f0905; width: 16px; height: 16px; border-radius: 50%; font-size: 9px; font-weight: 800; display: flex; align-items: center; justify-content: center; }

/* Slide Panel */
.slide-panel { width: 300px; height: 100vh; background: #16100c; border-right: 1px solid rgba(201,168,76,0.12); display: flex; flex-direction: column; overflow: hidden; }
.panel-header { display: flex; justify-content: space-between; align-items: center; padding: 16px 16px; border-bottom: 1px solid rgba(201,168,76,0.1); flex-shrink: 0; }
.panel-header h3 { color: #fdf6e3; font-size: 14px; font-weight: 700; }
.panel-close { background: none; border: none; cursor: pointer; color: #6b5a4e; transition: color 0.2s; padding: 4px; }
.panel-close:hover { color: #c9a84c; }
.panel-content { flex: 1; overflow-y: auto; padding: 16px; display: flex; flex-direction: column; gap: 12px; }
.panel-desc { color: #8b7355; font-size: 12px; line-height: 1.5; }
.panel-empty { color: #6b5a4e; font-size: 13px; text-align: center; padding: 32px 0; }
.panel-loading { color: #8b7355; font-size: 13px; text-align: center; padding: 20px 0; }

/* Search */
.panel-search-bar { display: flex; gap: 8px; }
.panel-search-bar input { flex: 1; background: rgba(255,255,255,0.05); border: 1px solid rgba(201,168,76,0.2); color: #fdf6e3; padding: 8px 12px; border-radius: 8px; font-size: 13px; outline: none; }
.panel-search-bar input:focus { border-color: #c9a84c; }
.panel-search-btn { background: #c9a84c; border: none; color: #0f0905; width: 36px; border-radius: 8px; cursor: pointer; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.panel-book-item { background: rgba(255,255,255,0.04); border: 1px solid rgba(201,168,76,0.1); border-radius: 10px; padding: 12px; }
.panel-book-title { color: #fdf6e3; font-size: 13px; font-weight: 700; margin-bottom: 4px; }
.panel-book-author { color: #8b7355; font-size: 12px; margin-bottom: 8px; }
.panel-book-link { color: #c9a84c; font-size: 12px; font-weight: 600; text-decoration: none; }
.panel-book-link:hover { text-decoration: underline; }

/* Forms */
.panel-form { display: flex; flex-direction: column; gap: 10px; }
.panel-input { background: rgba(255,255,255,0.05); border: 1px solid rgba(201,168,76,0.2); color: #fdf6e3; padding: 9px 12px; border-radius: 8px; font-size: 13px; outline: none; font-family: inherit; width: 100%; }
.panel-input:focus { border-color: #c9a84c; }
.panel-textarea { background: rgba(255,255,255,0.05); border: 1px solid rgba(201,168,76,0.2); color: #fdf6e3; padding: 9px 12px; border-radius: 8px; font-size: 13px; outline: none; font-family: inherit; width: 100%; resize: none; }
.panel-submit-btn { background: #c9a84c; color: #0f0905; border: none; padding: 10px; border-radius: 8px; font-size: 13px; font-weight: 700; cursor: pointer; transition: all 0.2s; }
.panel-submit-btn:hover { background: #e0c060; }
.panel-submit-btn:disabled { opacity: 0.6; cursor: not-allowed; }
.panel-msg { padding: 8px 12px; border-radius: 8px; font-size: 12px; font-weight: 600; }
.panel-msg.success { background: rgba(22,163,74,0.15); border: 1px solid rgba(22,163,74,0.3); color: #4ade80; }
.panel-msg.error { background: rgba(220,38,38,0.15); border: 1px solid rgba(220,38,38,0.3); color: #f87171; }
.cit-result-box { background: rgba(201,168,76,0.08); border: 1px solid rgba(201,168,76,0.2); border-radius: 8px; padding: 12px; }
.cit-result-text { color: #fdf6e3; font-size: 13px; line-height: 1.6; margin-bottom: 8px; font-style: italic; }
.copy-cit { background: #c9a84c; color: #0f0905; border: none; padding: 5px 12px; border-radius: 6px; font-size: 12px; font-weight: 700; cursor: pointer; }

/* My Requests */
.req-card { background: rgba(255,255,255,0.04); border: 1px solid rgba(201,168,76,0.1); border-radius: 10px; padding: 12px; }
.req-card-title { color: #fdf6e3; font-size: 13px; font-weight: 700; margin-bottom: 3px; }
.req-card-author { color: #8b7355; font-size: 12px; margin-bottom: 8px; }
.req-card-bottom { display: flex; justify-content: space-between; align-items: center; }
.req-status { padding: 2px 8px; border-radius: 4px; font-size: 11px; font-weight: 700; text-transform: uppercase; }
.req-status.pending { background: rgba(234,179,8,0.15); color: #fbbf24; }
.req-status.approved { background: rgba(22,163,74,0.15); color: #4ade80; }
.req-status.rejected { background: rgba(220,38,38,0.15); color: #f87171; }
.req-status.ordered { background: rgba(201,168,76,0.15); color: #c9a84c; }
.req-date { font-size: 11px; color: #6b5a4e; }
.req-note { margin-top: 8px; font-size: 12px; color: #c9a84c; background: rgba(201,168,76,0.08); padding: 6px 8px; border-radius: 6px; }

/* Saved */
.saved-card { background: rgba(255,255,255,0.04); border: 1px solid rgba(201,168,76,0.1); border-radius: 10px; padding: 12px; }
.saved-card-text { color: #c4aa94; font-size: 12px; line-height: 1.5; max-height: 80px; overflow: hidden; margin-bottom: 8px; }
.saved-card-bottom { display: flex; justify-content: space-between; align-items: center; }
.saved-time { font-size: 11px; color: #6b5a4e; }
.remove-saved { background: none; border: none; color: #ef4444; font-size: 11px; font-weight: 600; cursor: pointer; }

/* Quick Links */
.links-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
.link-card { background: rgba(255,255,255,0.04); border: 1px solid rgba(201,168,76,0.1); border-radius: 10px; padding: 14px 10px; text-decoration: none; transition: all 0.2s; display: block; }
.link-card:hover { border-color: #c9a84c; background: rgba(201,168,76,0.08); transform: translateY(-2px); }
.link-icon { font-size: 22px; margin-bottom: 6px; }
.link-name { color: #fdf6e3; font-size: 12px; font-weight: 700; margin-bottom: 3px; }
.link-desc { color: #6b5a4e; font-size: 11px; }

/* Stats */
.stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
.stat-card { background: rgba(255,255,255,0.04); border: 1px solid rgba(201,168,76,0.1); border-radius: 10px; padding: 16px 12px; text-align: center; }
.stat-card-num { font-size: 28px; font-weight: 900; color: #c9a84c; }
.stat-card-label { font-size: 11px; color: #6b5a4e; margin-top: 4px; }

/* Transition */
.panel-slide-enter-active, .panel-slide-leave-active { transition: all 0.25s ease; }
.panel-slide-enter-from, .panel-slide-leave-to { transform: translateX(-100%); opacity: 0; }
</style>