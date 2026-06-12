<template>
  <div class="admin-container">
    <div class="admin-header">
      <div class="header-left">
        <button @click="$emit('back')" class="back-btn">Back to Chat</button>
        <div>
          <h1>Admin Panel</h1>
          <p class="header-sub">Covenant University Library Management</p>
        </div>
      </div>
      <div class="admin-tabs">
        <button @click="activeTab = 'dashboard'" :class="{ active: activeTab === 'dashboard' }">Dashboard</button>
        <button @click="activeTab = 'settings'" :class="{ active: activeTab === 'settings' }">Settings</button>
        <button @click="activeTab = 'knowledge'" :class="{ active: activeTab === 'knowledge' }">Knowledge Base</button>
        <button @click="activeTab = 'personality'" :class="{ active: activeTab === 'personality' }">AI Personality</button>
        <button @click="activeTab = 'books'" :class="{ active: activeTab === 'books' }">Books</button>
        <button @click="activeTab = 'users'" :class="{ active: activeTab === 'users' }">Users</button>
        <button @click="activeTab = 'logs'" :class="{ active: activeTab === 'logs' }">Conversation Logs</button>
      </div>
    </div>

    <div v-if="activeTab === 'dashboard'" class="tab-content">
      <div class="tab-toolbar">
        <div>
          <h2>Dashboard</h2>
          <p class="tab-subtitle">Overview of your library chatbot system</p>
        </div>
        <button @click="loadDashboard" class="import-btn">Refresh</button>
      </div>
      <div v-if="isLoadingDashboard" class="loading">Loading dashboard...</div>
      <div v-else>
        <div class="stats-grid">
          <div class="stat-card" style="cursor:pointer" @click="goToTab('users')">
            <div class="stat-icon">👥</div>
            <div class="stat-info">
              <div class="stat-number">{{ stats.total_users }}</div>
              <div class="stat-label">Total Users</div>
            </div>
          </div>
          <div class="stat-card" style="cursor:pointer" @click="goToTab('users')">
            <div class="stat-icon">🎓</div>
            <div class="stat-info">
              <div class="stat-number">{{ stats.total_students }}</div>
              <div class="stat-label">Students</div>
            </div>
          </div>
          <div class="stat-card" style="cursor:pointer" @click="goToTab('logs')">
            <div class="stat-icon">💬</div>
            <div class="stat-info">
              <div class="stat-number">{{ stats.total_messages }}</div>
              <div class="stat-label">Total Messages</div>
            </div>
          </div>
          <div class="stat-card" style="cursor:pointer" @click="goToTab('logs')">
            <div class="stat-icon">🗂</div>
            <div class="stat-info">
              <div class="stat-number">{{ stats.total_sessions }}</div>
              <div class="stat-label">Conversations</div>
            </div>
          </div>
          <div class="stat-card" style="cursor:pointer" @click="goToTab('books')">
            <div class="stat-icon">📚</div>
            <div class="stat-info">
              <div class="stat-number">{{ stats.total_books }}</div>
              <div class="stat-label">Books in Catalogue</div>
            </div>
          </div>
          <div class="stat-card" style="cursor:pointer" @click="goToTab('books')">
            <div class="stat-icon">🆕</div>
            <div class="stat-info">
              <div class="stat-number">{{ stats.new_arrivals }}</div>
              <div class="stat-label">New Arrivals</div>
            </div>
          </div>
          <div class="stat-card" style="cursor:pointer" @click="goToTab('knowledge')">
            <div class="stat-icon">🧠</div>
            <div class="stat-info">
              <div class="stat-number">{{ stats.knowledge_entries }}</div>
              <div class="stat-label">Knowledge Entries</div>
            </div>
          </div>
          <div class="stat-card" style="cursor:pointer" @click="goToTab('logs')">
            <div class="stat-icon">📅</div>
            <div class="stat-info">
              <div class="stat-number">{{ stats.messages_today }}</div>
              <div class="stat-label">Messages Today</div>
            </div>
          </div>
        </div>
        <div class="dashboard-bottom">
          <div class="recent-card">
            <h3>Recent Conversations</h3>
            <div v-if="!stats.recent_sessions || stats.recent_sessions.length === 0" class="empty-state" style="padding: 20px">No conversations yet</div>
            <div v-else>
              <div v-for="session in stats.recent_sessions" :key="session.session_id" class="recent-session">
                <div class="recent-session-user">{{ session.user_name || 'Unknown' }}</div>
                <div class="recent-session-msg">{{ session.first_message }}</div>
                <div class="recent-session-time">{{ formatDateTime(session.started_at) }}</div>
              </div>
            </div>
          </div>
          <div class="recent-card">
            <h3>Quick Actions</h3>
            <div class="quick-action-list">
              <button @click="goToTab('books')" class="quick-action-btn">📚 Manage Books</button>
              <button @click="goToTab('knowledge')" class="quick-action-btn">🧠 Update Knowledge Base</button>
              <button @click="goToTab('settings')" class="quick-action-btn">⚙️ Library Settings</button>
              <button @click="goToTab('users')" class="quick-action-btn">👥 Manage Users</button>
              <button @click="goToTab('logs')" class="quick-action-btn">💬 View Conversations</button>
              <button @click="goToTab('personality')" class="quick-action-btn">🤖 AI Personality</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="activeTab === 'settings'" class="tab-content">
      <div v-if="isLoadingSettings" class="loading">Loading settings...</div>
      <div v-else class="settings-grid">
        <div class="settings-card">
          <h3>Library Hours</h3>
          <div v-for="setting in hourSettings" :key="setting.key" class="setting-item">
            <label>{{ setting.label }}</label>
            <div class="setting-row">
              <input v-model="setting.value" type="text" />
              <button @click="saveSetting(setting)" class="save-btn" :disabled="setting.saving">{{ setting.saving ? '...' : 'Save' }}</button>
            </div>
            <span v-if="setting.saved" class="saved-msg">Saved!</span>
          </div>
        </div>
        <div class="settings-card">
          <h3>Borrowing Policy</h3>
          <div v-for="setting in policySettings" :key="setting.key" class="setting-item">
            <label>{{ setting.label }}</label>
            <div class="setting-row">
              <input v-model="setting.value" type="text" />
              <button @click="saveSetting(setting)" class="save-btn" :disabled="setting.saving">{{ setting.saving ? '...' : 'Save' }}</button>
            </div>
            <span v-if="setting.saved" class="saved-msg">Saved!</span>
          </div>
        </div>
        <div class="settings-card">
          <h3>Contact Information</h3>
          <div v-for="setting in contactSettings" :key="setting.key" class="setting-item">
            <label>{{ setting.label }}</label>
            <div class="setting-row">
              <input v-model="setting.value" type="text" />
              <button @click="saveSetting(setting)" class="save-btn" :disabled="setting.saving">{{ setting.saving ? '...' : 'Save' }}</button>
            </div>
            <span v-if="setting.saved" class="saved-msg">Saved!</span>
          </div>
        </div>
        <div class="settings-card full-width">
          <h3>Announcement Banner</h3>
          <p class="card-subtitle">This message will show as a banner to all users in the chatbot</p>
          <div v-for="setting in announcementSettings" :key="setting.key" class="setting-item">
            <textarea v-model="setting.value" rows="3"></textarea>
            <div class="setting-row" style="margin-top: 8px">
              <button @click="saveSetting(setting)" class="save-btn wide" :disabled="setting.saving">{{ setting.saving ? 'Saving...' : 'Save Announcement' }}</button>
            </div>
            <span v-if="setting.saved" class="saved-msg">Saved!</span>
          </div>
        </div>
      </div>
    </div>

    <div v-if="activeTab === 'knowledge'" class="tab-content">
      <div class="tab-toolbar">
        <div>
          <h2>Knowledge Base</h2>
          <p class="tab-subtitle">Facts and FAQs the AI uses — {{ knowledge.length }} entries</p>
        </div>
        <button @click="showAddKnowledge = true" class="add-btn">+ Add Entry</button>
      </div>
      <div v-if="showAddKnowledge" class="modal-overlay" @click.self="cancelKnowledge">
        <div class="modal">
          <h3>{{ editingKnowledge ? 'Edit Entry' : 'Add Entry' }}</h3>
          <div class="modal-form">
            <div class="input-group">
              <label>Category</label>
              <select v-model="knowledgeForm.category">
                <option value="general">General</option>
                <option value="facilities">Facilities</option>
                <option value="databases">Databases and E-Resources</option>
                <option value="policies">Policies and Rules</option>
                <option value="services">Services</option>
                <option value="faq">FAQ</option>
                <option value="contacts">Contacts</option>
              </select>
            </div>
            <div class="input-group">
              <label>What should the AI know?</label>
              <p style="color:#8b7355;font-size:12px;margin:0 0 8px;">Write anything in plain English — facts, policies, hours, contacts, etc.</p>
              <textarea v-model="knowledgeForm.answer" rows="8" placeholder="e.g. The library has 4 floors. The ground floor has fiction books. The first floor has academic textbooks. The second floor is a quiet study area. Late returns attract a fine of N50 per day per book."></textarea>
            </div>
          </div>
          <div class="modal-actions">
            <button @click="cancelKnowledge" class="cancel-btn">Cancel</button>
            <button @click="saveKnowledge" class="save-btn" :disabled="isSavingKnowledge">{{ isSavingKnowledge ? 'Saving...' : editingKnowledge ? 'Update' : 'Save' }}</button>
          </div>
        </div>
      </div>
      <div v-if="isLoadingKnowledge" class="loading">Loading...</div>
      <div v-else-if="knowledge.length === 0" class="empty-state">No entries yet. Click Add Entry!</div>
      <div v-else class="table-wrapper">
        <table class="data-table">
          <thead>
            <tr><th>#</th><th>Category</th><th>Question</th><th>Answer</th><th>Status</th><th>Date</th><th>Actions</th></tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in knowledge" :key="item.id" :class="{ 'inactive-row': !item.active }">
              <td class="row-num">{{ index + 1 }}</td>
              <td><span class="category-badge">{{ item.category }}</span></td>
              <td class="question-cell">{{ item.question || '-' }}</td>
              <td class="answer-cell">{{ item.answer }}</td>
              <td><span class="status-badge" :class="item.active ? 'available' : 'unavailable'">{{ item.active ? 'Active' : 'Inactive' }}</span></td>
              <td class="date-cell">{{ formatDate(item.created_at) }}</td>
              <td>
                <div class="action-btns">
                  <button @click="toggleKnowledge(item)" class="toggle-btn" :class="{ active: item.active }">{{ item.active ? 'Disable' : 'Enable' }}</button>
                  <button @click="editKnowledge(item)" class="edit-btn">Edit</button>
                  <button @click="deleteKnowledge(item.id)" class="delete-btn">Delete</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div v-if="activeTab === 'personality'" class="tab-content">
      <div class="tab-toolbar">
        <div>
          <h2>AI Personality</h2>
          <p class="tab-subtitle">Customize how the chatbot presents itself to users</p>
        </div>
        <button @click="savePersonality" class="add-btn" :disabled="isSavingPersonality">{{ isSavingPersonality ? 'Saving...' : 'Save All Changes' }}</button>
      </div>
      <div v-if="personalitySaved" class="success-banner">All personality settings saved successfully!</div>
      <div class="settings-grid">
        <div class="settings-card">
          <h3>Chatbot Identity</h3>
          <div class="setting-item">
            <label>Chatbot Name</label>
            <input v-model="personality.ai_name" type="text" placeholder="e.g. COLRIS Library Assistant" class="full-input" />
            <p class="field-hint">This is how the AI identifies itself to users</p>
          </div>
          <div class="setting-item">
            <label>Greeting Message</label>
            <textarea v-model="personality.ai_greeting" rows="3" placeholder="e.g. Hello! I am the COLRIS Library AI Assistant..." class="full-input"></textarea>
            <p class="field-hint">First message users see when they open the chat</p>
          </div>
        </div>
        <div class="settings-card">
          <h3>Communication Style</h3>
          <div class="setting-item">
            <label>Tone</label>
            <select v-model="personality.ai_tone" class="form-select">
              <option value="professional and helpful">Professional and Helpful</option>
              <option value="friendly and casual">Friendly and Casual</option>
              <option value="formal and academic">Formal and Academic</option>
              <option value="warm and encouraging">Warm and Encouraging</option>
            </select>
            <p class="field-hint">How the AI communicates with users</p>
          </div>
        </div>
        <div class="settings-card full-width">
          <h3>Topic Restrictions</h3>
          <div class="setting-item">
            <label>Topics to Avoid</label>
            <textarea v-model="personality.ai_restrictions" rows="4" placeholder="e.g. Do not discuss politics..." class="full-input"></textarea>
            <p class="field-hint">List topics the AI should avoid. One per line.</p>
          </div>
        </div>
      </div>
    </div>

    <div v-if="activeTab === 'books'" class="tab-content">
      <div class="tab-toolbar">
        <div>
          <h2>Library Books</h2>
          <p class="tab-subtitle">{{ books.length }} books · <span class="new-arrival-count">{{ newArrivalCount }} new arrivals</span></p>
        </div>
        <div class="toolbar-actions">
          <button @click="showImportModal = true" class="import-btn">Import CSV</button>
          <button @click="showAddBook = true" class="add-btn">+ Add Book</button>
        </div>
      </div>
      <div v-if="showImportModal" class="modal-overlay" @click.self="closeImportModal">
        <div class="modal">
          <h3>Import Books from CSV</h3>
          <p class="modal-subtitle">Upload a CSV file to bulk import books</p>
          <div class="csv-template">
            <p class="template-label">Required CSV column order:</p>
            <code>Title, Author, ISBN, Category, Stock, Location</code>
          </div>
          <div class="input-group" style="margin-top: 20px">
            <label>Select CSV File</label>
            <input type="file" accept=".csv" @change="handleCsvFile" class="file-input" />
          </div>
          <div v-if="importResult" class="import-result" :class="importResult.errors > 0 ? 'has-errors' : 'success'">{{ importResult.message }}</div>
          <div class="modal-actions">
            <button @click="closeImportModal" class="cancel-btn">Close</button>
            <button @click="importCsv" class="save-btn" :disabled="!csvFile || isImporting">{{ isImporting ? 'Importing...' : 'Import Books' }}</button>
          </div>
        </div>
      </div>
      <div v-if="showAddBook" class="modal-overlay" @click.self="cancelBook">
        <div class="modal">
          <h3>{{ editingBook ? 'Edit Book' : 'Add New Book' }}</h3>
          <div class="modal-form">
            <div class="input-group"><label>Title</label><input v-model="bookForm.title" type="text" placeholder="Book title" /></div>
            <div class="input-group"><label>Author</label><input v-model="bookForm.author" type="text" placeholder="Author name" /></div>
            <div class="input-group"><label>ISBN</label><input v-model="bookForm.isbn" type="text" placeholder="ISBN number" /></div>
            <div class="input-group"><label>Category</label><input v-model="bookForm.category" type="text" placeholder="e.g. Computer Science" /></div>
            <div class="input-group"><label>Stock</label><input v-model="bookForm.stock" type="number" min="0" placeholder="0" /></div>
            <div class="input-group"><label>Shelf Location</label><input v-model="bookForm.location" type="text" placeholder="e.g. Section A, Shelf 3" /></div>
            <div class="input-group">
              <label class="checkbox-label">
                <input type="checkbox" v-model="bookForm.is_new_arrival" class="checkbox-input" />
                <span class="checkbox-text">Mark as New Arrival</span>
              </label>
            </div>
          </div>
          <div class="modal-actions">
            <button @click="cancelBook" class="cancel-btn">Cancel</button>
            <button @click="saveBook" class="save-btn" :disabled="isSavingBook">{{ isSavingBook ? 'Saving...' : editingBook ? 'Update Book' : 'Add Book' }}</button>
          </div>
        </div>
      </div>
      <div v-if="isLoadingBooks" class="loading">Loading books...</div>
      <div v-else-if="books.length === 0" class="empty-state">No books yet. Click Add Book or Import CSV!</div>
      <div v-else class="table-wrapper">
        <table class="data-table">
          <thead>
            <tr><th>#</th><th>Title</th><th>Author</th><th>Category</th><th>Stock</th><th>Location</th><th>Status</th><th>New Arrival</th><th>Actions</th></tr>
          </thead>
          <tbody>
            <tr v-for="(book, index) in books" :key="book.id">
              <td class="row-num">{{ index + 1 }}</td>
              <td><strong>{{ book.title }}</strong></td>
              <td>{{ book.author }}</td>
              <td>{{ book.category || '-' }}</td>
              <td>{{ book.stock }}</td>
              <td>{{ book.location || '-' }}</td>
              <td><span class="status-badge" :class="book.stock > 0 ? 'available' : 'unavailable'">{{ book.stock > 0 ? 'Available' : 'Out of Stock' }}</span></td>
              <td>
                <span v-if="book.is_new_arrival" class="new-arrival-badge">NEW</span>
                <span v-else class="no-arrival-text">—</span>
              </td>
              <td>
                <div class="action-btns">
                  <button @click="toggleArrival(book)" class="arrival-toggle-btn" :class="{ 'is-arrival': book.is_new_arrival }">{{ book.is_new_arrival ? 'Remove New' : 'Mark New' }}</button>
                  <button @click="editBook(book)" class="edit-btn">Edit</button>
                  <button @click="deleteBook(book.id)" class="delete-btn">Delete</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div v-if="activeTab === 'users'" class="tab-content">
      <div class="tab-toolbar">
        <h2>Registered Users</h2>
        <span class="user-count">{{ users.length }} users</span>
      </div>
      <div v-if="isLoadingUsers" class="loading">Loading users...</div>
      <div v-else class="table-wrapper">
        <table class="data-table">
          <thead>
            <tr><th>#</th><th>Name</th><th>Email</th><th>Role</th><th>Joined</th><th>Actions</th></tr>
          </thead>
          <tbody>
            <tr v-for="(user, index) in users" :key="user.id">
              <td class="row-num">{{ index + 1 }}</td>
              <td><strong>{{ user.name }}</strong></td>
              <td>{{ user.email }}</td>
              <td><span class="role-badge" :class="user.role">{{ user.role }}</span></td>
              <td>{{ formatDate(user.created_at) }}</td>
              <td>
                <div class="action-btns">
                  <select @change="updateRole(user, $event)" class="role-select">
                    <option value="student" :selected="user.role === 'student'">Student</option>
                    <option value="admin" :selected="user.role === 'admin'">Admin</option>
                  </select>
                  <button @click="deleteUser(user.id)" class="delete-btn">Delete</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div v-if="activeTab === 'logs'" class="tab-content">
      <div class="tab-toolbar">
        <div>
          <h2>Conversation Logs</h2>
          <p class="tab-subtitle">{{ totalMessages }} messages across {{ groupedSessions.length }} sessions</p>
        </div>
        <div class="toolbar-actions">
          <input v-model="logSearch" type="text" placeholder="Search..." class="search-input" />
          <button @click="exportLogs" class="export-btn">Export CSV</button>
          <button @click="showDeleteLogsModal = true" class="danger-btn">Delete Old</button>
          <button @click="loadLogs" class="import-btn">Refresh</button>
        </div>
      </div>
      <div v-if="showDeleteLogsModal" class="modal-overlay" @click.self="showDeleteLogsModal = false">
        <div class="modal">
          <h3>Delete Old Conversations</h3>
          <p class="modal-subtitle">Permanently delete conversations older than the selected period.</p>
          <div class="input-group">
            <label>Delete conversations older than</label>
            <select v-model="deleteOlderThan" class="form-select-modal">
              <option value="7">7 days</option>
              <option value="30">30 days</option>
              <option value="60">60 days</option>
              <option value="90">90 days</option>
              <option value="180">6 months</option>
            </select>
          </div>
          <div v-if="deleteLogsResult" class="import-result" :class="deleteLogsResult.success ? 'success' : 'has-errors'">{{ deleteLogsResult.message }}</div>
          <div class="modal-actions">
            <button @click="showDeleteLogsModal = false" class="cancel-btn">Cancel</button>
            <button @click="deleteOldLogs" class="danger-btn-modal" :disabled="isDeletingLogs">{{ isDeletingLogs ? 'Deleting...' : 'Delete Conversations' }}</button>
          </div>
        </div>
      </div>
      <div v-if="isLoadingLogs" class="loading">Loading logs...</div>
      <div v-else-if="filteredLogs.length === 0" class="empty-state">{{ logSearch ? 'No results found.' : 'No conversations yet.' }}</div>
      <div v-else>
        <div v-for="session in groupedSessions" :key="session.session_id" class="session-card">
          <div class="session-header" @click="toggleSession(session.session_id)">
            <div class="session-info">
              <div class="session-user">
                <span class="user-avatar-log">{{ getInitials(session.user_name) }}</span>
                <div>
                  <strong>{{ session.user_name || 'Unknown User' }}</strong>
                  <span class="session-email">{{ session.user_email || 'No email' }}</span>
                </div>
              </div>
              <div class="session-meta">
                <span class="msg-count-badge">{{ session.messages.length }} messages</span>
                <span class="date-cell">{{ formatDateTime(session.last_message) }}</span>
                <span class="chevron" :class="{ open: openSessions.includes(session.session_id) }">▾</span>
              </div>
            </div>
          </div>
          <div v-if="openSessions.includes(session.session_id)" class="session-messages">
            <div v-for="msg in session.messages" :key="msg.id" class="log-message" :class="msg.sender">
              <div class="msg-sender-label">{{ msg.sender === 'user' ? session.user_name || 'User' : 'Bot' }}</div>
              <div class="msg-bubble">{{ msg.message }}</div>
              <div class="msg-time">{{ formatDateTime(msg.created_at) }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Admin',
  emits: ['back'],
  data() {
    return {
      activeTab: 'dashboard',
      isLoadingDashboard: false,
      isLoadingSettings: true,
      isLoadingBooks: false,
      isLoadingUsers: false,
      isLoadingKnowledge: false,
      isLoadingLogs: false,
      stats: {
        total_users: 0, total_students: 0, total_messages: 0,
        total_sessions: 0, total_books: 0, new_arrivals: 0,
        knowledge_entries: 0, messages_today: 0, recent_sessions: [],
      },
      settings: [],
      books: [],
      users: [],
      knowledge: [],
      logs: [],
      logSearch: '',
      openSessions: [],
      showDeleteLogsModal: false,
      deleteOlderThan: '30',
      deleteLogsResult: null,
      isDeletingLogs: false,
      personality: {
        ai_name: '',
        ai_greeting: '',
        ai_tone: 'professional and helpful',
        ai_restrictions: '',
      },
      isSavingPersonality: false,
      personalitySaved: false,
      showAddBook: false,
      editingBook: null,
      isSavingBook: false,
      bookForm: { title: '', author: '', isbn: '', category: '', stock: 0, location: '', is_new_arrival: false },
      showAddKnowledge: false,
      editingKnowledge: null,
      isSavingKnowledge: false,
      knowledgeForm: { category: 'general', question: '', answer: '' },
      showImportModal: false,
      csvFile: null,
      isImporting: false,
      importResult: null,
    }
  },
  computed: {
    hourSettings() {
      return this.settings.filter(function(s) { return s.key.indexOf('hours_') === 0 })
    },
    policySettings() {
      return this.settings.filter(function(s) { return s.key === 'borrowing_limit' || s.key === 'fine_per_day' })
    },
    contactSettings() {
      return this.settings.filter(function(s) { return s.key.indexOf('contact_') === 0 })
    },
    announcementSettings() {
      return this.settings.filter(function(s) { return s.key === 'library_announcement' })
    },
    newArrivalCount() {
      return this.books.filter(function(b) { return b.is_new_arrival }).length
    },
    totalMessages() { return this.logs.length },
    filteredLogs() {
      if (!this.logSearch) return this.logs
      var q = this.logSearch.toLowerCase()
      return this.logs.filter(function(l) {
        return (l.user_name && l.user_name.toLowerCase().indexOf(q) !== -1) ||
               (l.user_email && l.user_email.toLowerCase().indexOf(q) !== -1) ||
               (l.message && l.message.toLowerCase().indexOf(q) !== -1)
      })
    },
    groupedSessions() {
      var map = {}
      for (var i = 0; i < this.filteredLogs.length; i++) {
        var msg = this.filteredLogs[i]
        var sid = msg.session_id || 'unknown'
        if (!map[sid]) {
          map[sid] = { session_id: sid, user_name: msg.user_name, user_email: msg.user_email, messages: [], last_message: msg.created_at }
        }
        map[sid].messages.push(msg)
        if (new Date(msg.created_at) > new Date(map[sid].last_message)) map[sid].last_message = msg.created_at
      }
      return Object.values(map).sort(function(a, b) { return new Date(b.last_message) - new Date(a.last_message) })
    },
  },
  watch: {
    activeTab: function(tab) {
      if (tab === 'dashboard') this.loadDashboard()
      if (tab === 'books' && this.books.length === 0) this.loadBooks()
      if (tab === 'users' && this.users.length === 0) this.loadUsers()
      if (tab === 'knowledge') this.loadKnowledge()
      if (tab === 'personality') this.loadPersonality()
      if (tab === 'logs') this.loadLogs()
    },
  },
  mounted: function() {
    this.loadSettings()
    this.loadDashboard()
  },
  methods: {
    token: function() { return localStorage.getItem('token') },
    headers: function() { return { Authorization: 'Bearer ' + this.token(), Accept: 'application/json' } },

    loadDashboard: async function() {
      this.isLoadingDashboard = true
      try {
        var response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/admin/dashboard', { headers: this.headers() })
        this.stats = response.data
      } catch(e) { console.error(e) } finally { this.isLoadingDashboard = false }
    },
    loadSettings: async function() {
      try {
        var response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/admin/settings', { headers: this.headers() })
        this.settings = response.data.settings.map(function(s) { return Object.assign({}, s, { saving: false, saved: false }) })
      } catch(e) { console.error(e) } finally { this.isLoadingSettings = false }
    },
    saveSetting: async function(setting) {
      setting.saving = true
      try {
        await axios.put('https://colris-chatbot-backend-production.up.railway.app/api/admin/settings/' + setting.key, { value: setting.value }, { headers: this.headers() })
        setting.saved = true
        setTimeout(function() { setting.saved = false }, 2000)
      } catch(e) { console.error(e) } finally { setting.saving = false }
    },
    loadBooks: async function() {
      this.isLoadingBooks = true
      try {
        var response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/admin/books', { headers: this.headers() })
        this.books = response.data.books
      } catch(e) { console.error(e) } finally { this.isLoadingBooks = false }
    },
    loadUsers: async function() {
      this.isLoadingUsers = true
      try {
        var response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/admin/users', { headers: this.headers() })
        this.users = response.data.users
      } catch(e) { console.error(e) } finally { this.isLoadingUsers = false }
    },
    loadKnowledge: async function() {
      this.isLoadingKnowledge = true
      try {
        var response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/admin/knowledge', { headers: this.headers() })
        this.knowledge = response.data.knowledge
      } catch(e) { console.error(e) } finally { this.isLoadingKnowledge = false }
    },
    loadPersonality: async function() {
      try {
        var response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/admin/personality', { headers: this.headers() })
        this.personality = {
          ai_name: response.data.ai_name || 'COLRIS Library Assistant',
          ai_greeting: response.data.ai_greeting || 'Hello! I am the COLRIS Library AI Assistant. How can I help you today?',
          ai_tone: response.data.ai_tone || 'professional and helpful',
          ai_restrictions: response.data.ai_restrictions || '',
        }
      } catch(e) { console.error(e) }
    },
    savePersonality: async function() {
      this.isSavingPersonality = true
      try {
        await axios.put('https://colris-chatbot-backend-production.up.railway.app/api/admin/personality', this.personality, { headers: this.headers() })
        this.personalitySaved = true
        setTimeout(function() { this.personalitySaved = false }.bind(this), 3000)
      } catch(e) { console.error(e) } finally { this.isSavingPersonality = false }
    },
    loadLogs: async function() {
      this.isLoadingLogs = true
      try {
        var response = await axios.get('https://colris-chatbot-backend-production.up.railway.app/api/admin/logs', { headers: this.headers() })
        this.logs = response.data.logs.data || response.data.logs
      } catch(e) { console.error(e) } finally { this.isLoadingLogs = false }
    },
    exportLogs: function() {
      if (this.logs.length === 0) { alert('No logs to export.'); return }
      var rows = [['Session ID', 'User Name', 'User Email', 'Sender', 'Message', 'Date']]
      for (var i = 0; i < this.logs.length; i++) {
        var log = this.logs[i]
        var msg = (log.message || '').replace(/"/g, '""')
        rows.push([log.session_id, log.user_name || '', log.user_email || '', log.sender, '"' + msg + '"', log.created_at])
      }
      var csv = rows.map(function(r) { return r.join(',') }).join('\n')
      var blob = new Blob([csv], { type: 'text/csv' })
      var url = URL.createObjectURL(blob)
      var a = document.createElement('a')
      a.href = url
      a.download = 'conversation_logs_' + new Date().toISOString().slice(0, 10) + '.csv'
      a.click()
      URL.revokeObjectURL(url)
    },
    deleteOldLogs: async function() {
      this.isDeletingLogs = true
      this.deleteLogsResult = null
      try {
        var response = await axios.delete('https://colris-chatbot-backend-production.up.railway.app/api/admin/logs/old', { headers: this.headers(), data: { days: this.deleteOlderThan } })
        this.deleteLogsResult = { success: true, message: response.data.message }
        await this.loadLogs()
      } catch(e) {
        this.deleteLogsResult = { success: false, message: 'Failed to delete logs. Please try again.' }
      } finally { this.isDeletingLogs = false }
    },
    toggleSession: function(sessionId) {
      var idx = this.openSessions.indexOf(sessionId)
      if (idx === -1) this.openSessions.push(sessionId)
      else this.openSessions.splice(idx, 1)
    },
    getInitials: function(name) {
      if (!name) return '?'
      return name.split(' ').map(function(n) { return n[0] }).join('').toUpperCase().slice(0, 2)
    },
    toggleArrival: async function(book) {
      try {
        await axios.put('https://colris-chatbot-backend-production.up.railway.app/api/admin/books/' + book.id + '/toggle-arrival', {}, { headers: this.headers() })
        book.is_new_arrival = !book.is_new_arrival
      } catch(e) { console.error(e) }
    },
    cancelKnowledge: function() {
      this.showAddKnowledge = false; this.editingKnowledge = null
      this.knowledgeForm = { category: 'general', question: '', answer: '' }
    },
    editKnowledge: function(item) {
      this.editingKnowledge = item
      this.knowledgeForm = { category: item.category, question: item.question || '', answer: item.answer }
      this.showAddKnowledge = true
    },
    saveKnowledge: async function() {
      if (!this.knowledgeForm.answer) return
      this.isSavingKnowledge = true
      try {
        if (this.editingKnowledge) {
          await axios.put('https://colris-chatbot-backend-production.up.railway.app/api/admin/knowledge/' + this.editingKnowledge.id, this.knowledgeForm, { headers: this.headers() })
        } else {
          await axios.post('https://colris-chatbot-backend-production.up.railway.app/api/admin/knowledge', this.knowledgeForm, { headers: this.headers() })
        }
        this.cancelKnowledge(); await this.loadKnowledge()
      } catch(e) { console.error(e) } finally { this.isSavingKnowledge = false }
    },
    toggleKnowledge: async function(item) {
      try {
        await axios.put('https://colris-chatbot-backend-production.up.railway.app/api/admin/knowledge/' + item.id + '/toggle', {}, { headers: this.headers() })
        item.active = !item.active
      } catch(e) { console.error(e) }
    },
    deleteKnowledge: async function(id) {
      if (!confirm('Delete this knowledge entry?')) return
      try {
        await axios.delete('https://colris-chatbot-backend-production.up.railway.app/api/admin/knowledge/' + id, { headers: this.headers() })
        await this.loadKnowledge()
      } catch(e) { console.error(e) }
    },
    resetBookForm: function() {
      this.bookForm = { title: '', author: '', isbn: '', category: '', stock: 0, location: '', is_new_arrival: false }
    },
    cancelBook: function() { this.showAddBook = false; this.editingBook = null; this.resetBookForm() },
    editBook: function(book) {
      this.editingBook = book
      this.bookForm = { title: book.title, author: book.author, isbn: book.isbn || '', category: book.category || '', stock: book.stock, location: book.location || '', is_new_arrival: !!book.is_new_arrival }
      this.showAddBook = true
    },
    saveBook: async function() {
      if (!this.bookForm.title || !this.bookForm.author) return
      this.isSavingBook = true
      try {
        if (this.editingBook) {
          await axios.put('https://colris-chatbot-backend-production.up.railway.app/api/admin/books/' + this.editingBook.id, this.bookForm, { headers: this.headers() })
        } else {
          await axios.post('https://colris-chatbot-backend-production.up.railway.app/api/admin/books', this.bookForm, { headers: this.headers() })
        }
        this.showAddBook = false; this.editingBook = null; this.resetBookForm(); await this.loadBooks()
      } catch(e) { console.error(e) } finally { this.isSavingBook = false }
    },
    deleteBook: async function(id) {
      if (!confirm('Delete this book?')) return
      try {
        await axios.delete('https://colris-chatbot-backend-production.up.railway.app/api/admin/books/' + id, { headers: this.headers() })
        await this.loadBooks()
      } catch(e) { console.error(e) }
    },
    closeImportModal: function() { this.showImportModal = false; this.importResult = null; this.csvFile = null },
    handleCsvFile: function(event) { this.csvFile = event.target.files[0]; this.importResult = null },
    importCsv: async function() {
      if (!this.csvFile) return
      this.isImporting = true
      try {
        var formData = new FormData()
        formData.append('csv_file', this.csvFile)
        var response = await axios.post('https://colris-chatbot-backend-production.up.railway.app/api/admin/books/import', formData, { headers: Object.assign({}, this.headers(), { 'Content-Type': 'multipart/form-data' }) })
        this.importResult = response.data; this.csvFile = null; await this.loadBooks()
      } catch(e) { console.error(e); this.importResult = { message: 'Import failed. Please check your CSV format.', errors: 1 } } finally { this.isImporting = false }
    },
    updateRole: async function(user, event) {
      try {
        await axios.put('https://colris-chatbot-backend-production.up.railway.app/api/admin/users/' + user.id + '/role', { role: event.target.value }, { headers: this.headers() })
        user.role = event.target.value
      } catch(e) { console.error(e) }
    },
    deleteUser: async function(id) {
      if (!confirm('Delete this user?')) return
      try {
        await axios.delete('https://colris-chatbot-backend-production.up.railway.app/api/admin/users/' + id, { headers: this.headers() })
        await this.loadUsers()
      } catch(e) { console.error(e) }
    },
    goToTab: function(tab) {
      this.activeTab = tab
    },
    formatDate: function(date) {
      return new Date(date).toLocaleDateString('en-GB', { day: 'numeric', month: 'short', year: 'numeric' })
    },
    formatDateTime: function(date) {
      return new Date(date).toLocaleString('en-GB', { day: 'numeric', month: 'short', year: 'numeric', hour: '2-digit', minute: '2-digit' })
    },
  },
}
</script>

<style scoped>
* { box-sizing: border-box; }

.admin-container { min-height: 100vh; background: #f0ece4; font-family: 'Segoe UI', Arial, sans-serif; animation: fadeIn 0.4s ease; }
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

/* ── HEADER ── */
.admin-header { background: #1a0f0a; padding: 0 36px; display: flex; align-items: center; justify-content: space-between; flex-wrap: wrap; gap: 0; min-height: 60px; position: sticky; top: 0; z-index: 50; box-shadow: 0 4px 24px rgba(0,0,0,0.35); }
.header-left { display: flex; align-items: center; gap: 20px; padding: 12px 0; }
.back-btn { background: rgba(201,168,76,0.15); border: 1px solid rgba(201,168,76,0.4); color: #c9a84c; padding: 6px 14px; border-radius: 6px; cursor: pointer; font-size: 12px; font-weight: 700; letter-spacing: 0.5px; transition: all 0.2s; text-transform: uppercase; }
.back-btn:hover { background: #c9a84c; color: #1a0f0a; }
.admin-header h1 { color: #ffffff; font-size: 17px; font-weight: 700; margin: 0; letter-spacing: 0.5px; }
.header-sub { color: rgba(255,255,255,0.4); font-size: 11px; margin: 2px 0 0; }
.admin-tabs { display: flex; align-self: stretch; }
.admin-tabs button { background: transparent; border: none; border-bottom: 2px solid transparent; color: rgba(255,255,255,0.5); padding: 0 16px; cursor: pointer; font-size: 12px; font-weight: 600; letter-spacing: 0.4px; transition: all 0.2s; height: 60px; text-transform: uppercase; white-space: nowrap; }
.admin-tabs button.active { color: #c9a84c; border-bottom-color: #c9a84c; }
.admin-tabs button:hover { color: #ffffff; background: rgba(255,255,255,0.05); }

/* ── TAB CONTENT ── */
.tab-content { padding: 28px 36px; animation: slideUp 0.25s ease; }
@keyframes slideUp { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

.tab-toolbar { display: flex; align-items: flex-start; justify-content: space-between; margin-bottom: 24px; flex-wrap: wrap; gap: 12px; }
.tab-toolbar h2 { color: #1a0f0a; font-size: 20px; font-weight: 700; margin: 0; letter-spacing: -0.3px; }
.tab-subtitle { color: #8b7355; font-size: 13px; margin: 4px 0 0; }
.new-arrival-count { color: #c9a84c; font-weight: 700; }
.toolbar-actions { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }

/* ── BUTTONS ── */
.add-btn { background: #2c1810; color: #fdf6e3; border: none; padding: 9px 20px; border-radius: 6px; cursor: pointer; font-size: 12px; font-weight: 700; letter-spacing: 0.4px; text-transform: uppercase; transition: all 0.2s; white-space: nowrap; box-shadow: 0 2px 8px rgba(44,24,16,0.25); }
.add-btn:hover { background: #1a0f0a; transform: translateY(-1px); box-shadow: 0 4px 16px rgba(44,24,16,0.35); }
.add-btn:disabled { opacity: 0.5; cursor: not-allowed; transform: none; }
.import-btn { background: white; color: #5c3d2e; border: 1.5px solid #d4c4b0; padding: 9px 20px; border-radius: 6px; cursor: pointer; font-size: 12px; font-weight: 700; letter-spacing: 0.4px; text-transform: uppercase; transition: all 0.2s; white-space: nowrap; }
.import-btn:hover { border-color: #c9a84c; color: #2c1810; background: #fdf6e3; }
.export-btn { background: #f0fdf4; color: #16a34a; border: 1.5px solid #bbf7d0; padding: 9px 20px; border-radius: 6px; cursor: pointer; font-size: 12px; font-weight: 700; letter-spacing: 0.4px; text-transform: uppercase; transition: all 0.2s; white-space: nowrap; }
.export-btn:hover { background: #16a34a; color: white; border-color: #16a34a; }
.danger-btn { background: white; color: #dc2626; border: 1.5px solid #fecaca; padding: 9px 20px; border-radius: 6px; cursor: pointer; font-size: 12px; font-weight: 700; letter-spacing: 0.4px; text-transform: uppercase; transition: all 0.2s; white-space: nowrap; }
.danger-btn:hover { background: #dc2626; color: white; }
.danger-btn-modal { background: #dc2626; color: white; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-size: 13px; font-weight: 600; transition: all 0.2s; }
.danger-btn-modal:hover { background: #b91c1c; }
.danger-btn-modal:disabled { opacity: 0.6; cursor: not-allowed; }
.save-btn { background: #2c1810; color: #fdf6e3; border: none; padding: 9px 18px; border-radius: 6px; font-size: 12px; font-weight: 700; letter-spacing: 0.4px; text-transform: uppercase; cursor: pointer; transition: all 0.2s; box-shadow: 0 2px 8px rgba(44,24,16,0.2); white-space: nowrap; }
.save-btn.wide { width: 100%; padding: 12px; }
.save-btn:hover { background: #1a0f0a; transform: translateY(-1px); }
.save-btn:disabled { opacity: 0.5; cursor: not-allowed; transform: none; }
.cancel-btn { background: #f5ede0; color: #8b7355; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-size: 13px; font-weight: 600; transition: all 0.2s; }
.cancel-btn:hover { background: #e8dcc8; color: #5c3d2e; }

/* ── STATS GRID ── */
.stats-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 16px; margin-bottom: 28px; }
.stat-card { background: white; border-radius: 12px; padding: 22px 20px; display: flex; align-items: center; gap: 16px; box-shadow: 0 1px 4px rgba(0,0,0,0.06), 0 4px 16px rgba(0,0,0,0.04); border: 1px solid rgba(0,0,0,0.06); transition: all 0.25s ease; position: relative; overflow: hidden; }
.stat-card::after { content: ""; position: absolute; top: 0; left: 0; width: 3px; height: 100%; background: #c9a84c; opacity: 0; transition: opacity 0.25s; }
.stat-card:hover { transform: translateY(-3px); box-shadow: 0 8px 28px rgba(0,0,0,0.1); }
.stat-card:hover::after { opacity: 1; }
.stat-icon { font-size: 28px; width: 52px; height: 52px; background: #fdf6e3; border-radius: 10px; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
.stat-number { font-size: 28px; font-weight: 800; color: #1a0f0a; line-height: 1; }
.stat-label { font-size: 12px; color: #8b7355; font-weight: 500; margin-top: 4px; text-transform: uppercase; letter-spacing: 0.5px; }

/* ── DASHBOARD BOTTOM ── */
.dashboard-bottom { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
.recent-card { background: white; border-radius: 12px; padding: 24px; box-shadow: 0 1px 4px rgba(0,0,0,0.06); border: 1px solid rgba(0,0,0,0.06); }
.recent-card h3 { color: #1a0f0a; font-size: 14px; font-weight: 700; margin: 0 0 18px; text-transform: uppercase; letter-spacing: 0.8px; border-bottom: 2px solid #f0ece4; padding-bottom: 12px; }
.recent-session { padding: 10px 0; border-bottom: 1px solid #f5ede0; transition: background 0.2s; cursor: default; }
.recent-session:last-child { border-bottom: none; }
.recent-session:hover { background: #fdf6e3; padding-left: 6px; border-radius: 4px; }
.recent-session-user { font-size: 13px; font-weight: 700; color: #1a0f0a; }
.recent-session-msg { font-size: 12px; color: #8b7355; margin: 3px 0; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.recent-session-time { font-size: 11px; color: #b8a898; }
.quick-action-list { display: flex; flex-direction: column; gap: 8px; }
.quick-action-btn { background: #faf7f2; border: 1px solid #e8dcc8; color: #2c1810; padding: 12px 16px; border-radius: 8px; cursor: pointer; font-size: 13px; font-weight: 600; text-align: left; transition: all 0.2s; display: flex; align-items: center; gap: 8px; }
.quick-action-btn:hover { background: #1a0f0a; color: white; border-color: #1a0f0a; transform: translateX(4px); }

/* ── SETTINGS ── */
.settings-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(340px, 1fr)); gap: 20px; }
.settings-card { background: white; border: 1px solid rgba(0,0,0,0.07); border-radius: 12px; padding: 24px; box-shadow: 0 1px 4px rgba(0,0,0,0.05); transition: box-shadow 0.25s; }
.settings-card:hover { box-shadow: 0 6px 24px rgba(0,0,0,0.08); }
.settings-card.full-width { grid-column: 1 / -1; }
.settings-card h3 { color: #1a0f0a; font-size: 13px; font-weight: 700; margin: 0 0 18px; text-transform: uppercase; letter-spacing: 0.8px; border-bottom: 2px solid #f0ece4; padding-bottom: 12px; }
.card-subtitle { color: #b8a898; font-size: 12px; margin: -12px 0 16px; }
.setting-item { margin-bottom: 18px; }
.setting-item label { display: block; color: #5c3d2e; font-size: 12px; font-weight: 700; margin-bottom: 8px; text-transform: uppercase; letter-spacing: 0.5px; }
.setting-row { display: flex; gap: 10px; }
.setting-row input, textarea { flex: 1; padding: 10px 14px; border: 1.5px solid #e2d8cc; border-radius: 6px; font-size: 14px; color: #1a0f0a; background: #faf7f2; outline: none; font-family: inherit; width: 100%; box-sizing: border-box; transition: all 0.2s; }
textarea { resize: vertical; }
.setting-row input:focus, textarea:focus { border-color: #c9a84c; background: white; box-shadow: 0 0 0 3px rgba(201,168,76,0.12); }
.full-input { width: 100%; padding: 10px 14px; border: 1.5px solid #e2d8cc; border-radius: 6px; font-size: 14px; color: #1a0f0a; background: #faf7f2; outline: none; font-family: inherit; box-sizing: border-box; transition: all 0.2s; }
.full-input:focus { border-color: #c9a84c; background: white; box-shadow: 0 0 0 3px rgba(201,168,76,0.12); }
.field-hint { color: #b8a898; font-size: 11px; margin: 6px 0 0; }
.form-select { width: 100%; padding: 10px 14px; border: 1.5px solid #e2d8cc; border-radius: 6px; font-size: 14px; color: #1a0f0a; background: #faf7f2; outline: none; cursor: pointer; transition: all 0.2s; }
.form-select:focus { border-color: #c9a84c; }
.form-select-modal { width: 100%; padding: 10px 14px; border: 1.5px solid #e2d8cc; border-radius: 6px; font-size: 14px; color: #1a0f0a; background: #faf7f2; outline: none; cursor: pointer; margin-top: 6px; }
.saved-msg { font-size: 12px; color: #16a34a; font-weight: 600; }
.success-banner { background: #f0fdf4; border: 1px solid #bbf7d0; color: #16a34a; padding: 12px 16px; border-radius: 8px; font-size: 13px; font-weight: 600; margin-bottom: 20px; text-align: center; }

/* ── TABLE ── */
.table-wrapper { background: white; border: 1px solid rgba(0,0,0,0.07); border-radius: 12px; overflow: hidden; overflow-x: auto; box-shadow: 0 1px 4px rgba(0,0,0,0.05); }
.data-table { width: 100%; border-collapse: collapse; min-width: 700px; }
.data-table th { background: #1a0f0a; color: rgba(255,255,255,0.7); font-size: 11px; font-weight: 700; padding: 13px 16px; text-align: left; text-transform: uppercase; letter-spacing: 0.8px; white-space: nowrap; }
.data-table td { padding: 14px 16px; border-bottom: 1px solid #f5ede0; font-size: 13px; color: #2c1810; vertical-align: middle; }
.data-table tr:last-child td { border-bottom: none; }
.data-table tr:hover td { background: #fdf6e3; }
.inactive-row td { opacity: 0.4; }
.row-num { color: #b8a898; font-size: 12px; font-weight: 600; width: 32px; }
.question-cell { max-width: 180px; font-size: 13px; color: #5c3d2e; font-weight: 500; line-height: 1.4; }
.answer-cell { max-width: 280px; font-size: 13px; color: #2c1810; line-height: 1.5; white-space: pre-wrap; word-break: break-word; }
.date-cell { white-space: nowrap; font-size: 12px; color: #8b7355; }

/* ── BADGES ── */
.category-badge { background: #fdf6e3; color: #8b5e3c; border: 1px solid #e8dcc8; padding: 3px 10px; border-radius: 4px; font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.5px; white-space: nowrap; }
.status-badge { padding: 4px 10px; border-radius: 4px; font-size: 11px; font-weight: 700; white-space: nowrap; text-transform: uppercase; letter-spacing: 0.4px; }
.status-badge.available { background: #f0fdf4; color: #16a34a; border: 1px solid #bbf7d0; }
.status-badge.unavailable { background: #fef2f2; color: #dc2626; border: 1px solid #fecaca; }
.role-badge { padding: 4px 10px; border-radius: 4px; font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.4px; }
.role-badge.admin { background: #1a0f0a; color: #c9a84c; }
.role-badge.student { background: #f0fdf4; color: #16a34a; border: 1px solid #bbf7d0; }
.new-arrival-badge { background: #c9a84c; color: white; padding: 3px 10px; border-radius: 4px; font-size: 11px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.5px; }
.no-arrival-text { color: #d1c4b0; }
.msg-count-badge { background: #fdf6e3; color: #8b5e3c; border: 1px solid #e8dcc8; padding: 3px 10px; border-radius: 4px; font-size: 11px; font-weight: 700; }
.user-count { background: #fdf6e3; color: #8b5e3c; padding: 4px 12px; border-radius: 4px; font-size: 12px; font-weight: 700; border: 1px solid #e8dcc8; text-transform: uppercase; letter-spacing: 0.4px; }

/* ── ACTION BUTTONS ── */
.action-btns { display: flex; gap: 6px; align-items: center; flex-wrap: wrap; }
.toggle-btn { padding: 5px 10px; border-radius: 4px; font-size: 11px; font-weight: 700; cursor: pointer; border: 1px solid #e8dcc8; background: white; color: #8b7355; transition: all 0.2s; white-space: nowrap; text-transform: uppercase; letter-spacing: 0.3px; }
.toggle-btn.active { background: #f0fdf4; color: #16a34a; border-color: #bbf7d0; }
.toggle-btn:hover { border-color: #c9a84c; color: #5c3d2e; }
.edit-btn { background: white; color: #5c3d2e; border: 1px solid #e8dcc8; padding: 5px 12px; border-radius: 4px; cursor: pointer; font-size: 11px; font-weight: 700; transition: all 0.2s; white-space: nowrap; text-transform: uppercase; letter-spacing: 0.3px; }
.edit-btn:hover { background: #2c1810; color: white; border-color: #2c1810; }
.delete-btn { background: white; color: #dc2626; border: 1px solid #fecaca; padding: 5px 12px; border-radius: 4px; cursor: pointer; font-size: 11px; font-weight: 700; transition: all 0.2s; white-space: nowrap; text-transform: uppercase; letter-spacing: 0.3px; }
.delete-btn:hover { background: #dc2626; color: white; }
.arrival-toggle-btn { padding: 5px 10px; border-radius: 4px; font-size: 11px; font-weight: 700; cursor: pointer; border: 1px solid #e8dcc8; background: white; color: #8b7355; transition: all 0.2s; white-space: nowrap; text-transform: uppercase; }
.arrival-toggle-btn.is-arrival { background: #1a0f0a; color: #c9a84c; border-color: #1a0f0a; }
.role-select { padding: 5px 10px; border: 1.5px solid #e8dcc8; border-radius: 4px; font-size: 12px; color: #2c1810; background: white; cursor: pointer; outline: none; }

/* ── SEARCH ── */
.search-input { padding: 9px 14px; border: 1.5px solid #e2d8cc; border-radius: 6px; font-size: 13px; color: #2c1810; background: white; outline: none; width: 220px; transition: all 0.2s; }
.search-input:focus { border-color: #c9a84c; box-shadow: 0 0 0 3px rgba(201,168,76,0.1); }

/* ── MISC ── */
.loading { text-align: center; padding: 60px; color: #8b7355; font-size: 14px; }
.empty-state { text-align: center; padding: 80px; color: #b8a898; font-size: 15px; }
.checkbox-label { display: flex !important; align-items: center; gap: 10px; cursor: pointer; }
.checkbox-input { width: 16px; height: 16px; accent-color: #c9a84c; cursor: pointer; }
.checkbox-text { font-size: 14px; color: #2c1810; font-weight: 600; }
.csv-template { background: #faf7f2; border: 1.5px solid #e8dcc8; border-radius: 8px; padding: 16px; margin-bottom: 8px; }
.template-label { color: #5c3d2e; font-size: 12px; font-weight: 700; margin-bottom: 6px; display: block; text-transform: uppercase; letter-spacing: 0.5px; }
.csv-template code { display: block; font-family: monospace; font-size: 12px; color: #2c1810; background: white; padding: 6px 10px; border-radius: 4px; border: 1px solid #e8dcc8; }
.file-input { width: 100%; padding: 10px; border: 2px dashed #e8dcc8; border-radius: 6px; font-size: 14px; color: #2c1810; background: #faf7f2; cursor: pointer; box-sizing: border-box; transition: border-color 0.2s; }
.file-input:hover { border-color: #c9a84c; }
.import-result { padding: 12px 16px; border-radius: 6px; font-size: 13px; font-weight: 600; margin-top: 12px; }
.import-result.success { background: #f0fdf4; border: 1px solid #bbf7d0; color: #16a34a; }
.import-result.has-errors { background: #fef9e7; border: 1px solid #fde68a; color: #92400e; }

/* ── MODAL ── */
.modal-overlay { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(0,0,0,0.6); z-index: 100; display: flex; align-items: center; justify-content: center; backdrop-filter: blur(4px); }
.modal { background: white; border-radius: 16px; padding: 32px; width: 100%; max-width: 520px; box-shadow: 0 24px 80px rgba(0,0,0,0.3); max-height: 90vh; overflow-y: auto; animation: modalIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
@keyframes modalIn { from { transform: scale(0.92) translateY(10px); opacity: 0; } to { transform: scale(1) translateY(0); opacity: 1; } }
.modal h3 { color: #1a0f0a; font-size: 18px; font-weight: 800; margin: 0 0 24px; border-bottom: 2px solid #f0ece4; padding-bottom: 14px; }
.modal-subtitle { color: #8b7355; font-size: 13px; margin: -18px 0 20px; }
.modal-form { display: flex; flex-direction: column; gap: 16px; }
.input-group label { display: block; color: #5c3d2e; font-size: 11px; font-weight: 700; margin-bottom: 6px; text-transform: uppercase; letter-spacing: 0.5px; }
.input-group input[type='text'], .input-group input[type='number'] { width: 100%; padding: 11px 14px; border: 1.5px solid #e2d8cc; border-radius: 6px; font-size: 14px; color: #1a0f0a; outline: none; box-sizing: border-box; background: #faf7f2; transition: all 0.2s; }
.input-group input:focus { border-color: #c9a84c; background: white; box-shadow: 0 0 0 3px rgba(201,168,76,0.12); }
.input-group textarea { width: 100%; padding: 11px 14px; border: 1.5px solid #e2d8cc; border-radius: 6px; font-size: 14px; color: #1a0f0a; outline: none; box-sizing: border-box; background: #faf7f2; font-family: inherit; resize: vertical; transition: all 0.2s; }
.input-group textarea:focus { border-color: #c9a84c; background: white; box-shadow: 0 0 0 3px rgba(201,168,76,0.12); }
.input-group select { width: 100%; padding: 11px 14px; border: 1.5px solid #e2d8cc; border-radius: 6px; font-size: 14px; color: #1a0f0a; outline: none; box-sizing: border-box; background: #faf7f2; cursor: pointer; }
.modal-actions { display: flex; gap: 12px; margin-top: 24px; justify-content: flex-end; }

/* ── CONVERSATION LOGS ── */
.session-card { background: white; border: 1px solid rgba(0,0,0,0.07); border-radius: 12px; margin-bottom: 10px; overflow: hidden; box-shadow: 0 1px 4px rgba(0,0,0,0.04); transition: all 0.2s; }
.session-card:hover { border-color: #c9a84c; box-shadow: 0 4px 16px rgba(0,0,0,0.08); }
.session-header { padding: 14px 20px; cursor: pointer; transition: background 0.2s; }
.session-header:hover { background: #fdf6e3; }
.session-info { display: flex; align-items: center; justify-content: space-between; flex-wrap: wrap; gap: 10px; }
.session-user { display: flex; align-items: center; gap: 12px; }
.user-avatar-log { width: 36px; height: 36px; border-radius: 8px; background: #1a0f0a; color: #c9a84c; display: flex; align-items: center; justify-content: center; font-size: 13px; font-weight: 800; flex-shrink: 0; }
.session-user strong { color: #1a0f0a; font-size: 14px; display: block; }
.session-email { color: #8b7355; font-size: 12px; }
.session-meta { display: flex; align-items: center; gap: 12px; }
.chevron { color: #8b7355; font-size: 18px; transition: transform 0.2s; display: inline-block; }
.chevron.open { transform: rotate(180deg); }
.session-messages { border-top: 1px solid #f0ece4; padding: 16px 20px; display: flex; flex-direction: column; gap: 10px; max-height: 400px; overflow-y: auto; background: #faf7f2; }
.log-message { display: flex; flex-direction: column; max-width: 75%; }
.log-message.user { align-self: flex-end; align-items: flex-end; }
.log-message.bot { align-self: flex-start; align-items: flex-start; }
.msg-sender-label { font-size: 10px; font-weight: 700; color: #8b7355; margin-bottom: 3px; text-transform: uppercase; letter-spacing: 0.5px; }
.msg-bubble { padding: 10px 14px; border-radius: 10px; font-size: 13px; line-height: 1.5; word-break: break-word; white-space: pre-wrap; }
.log-message.user .msg-bubble { background: #1a0f0a; color: #fdf6e3; border-bottom-right-radius: 3px; }
.log-message.bot .msg-bubble { background: white; color: #2c1810; border: 1px solid #e8dcc8; border-bottom-left-radius: 3px; }
.msg-time { font-size: 10px; color: #b8a898; margin-top: 3px; }
</style>