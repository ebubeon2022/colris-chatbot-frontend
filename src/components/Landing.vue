<template>
  <div class="landing">
    <canvas ref="particles" class="particles-canvas"></canvas>
    <nav class="nav">
      <div class="nav-brand">
        <img src="/cu-logo.png" alt="Covenant University" class="cu-logo" />
        <div class="nav-brand-text">
          <span class="nav-title">COLRIS AI Assistant</span>
          <span class="nav-sub">Covenant University Library</span>
        </div>
      </div>
      <div class="nav-actions">
        <button @click="$emit('show-login')" class="nav-login">Sign In</button>
        <button @click="$emit('show-register')" class="nav-register">Get Started</button>
      </div>
    </nav>

    <section class="hero">
      <div class="hero-inner">
        <div class="hero-badge">Covenant University · Ota, Ogun State</div>
        <h1 class="hero-title">{{ typedText }}<span class="cursor">|</span><br><span class="gold">Assistant</span></h1>
        <p class="hero-sub">Instantly find books, generate citations, request resources, and get library support — all through a smart conversational interface built for CU students.</p>
        <div class="hero-actions">
          <button @click="$emit('show-register')" class="btn-primary">Get Started Free</button>
          <button @click="$emit('show-login')" class="btn-secondary">Sign In</button>
        </div>
        <div class="hero-stats">
          <div class="stat"><span class="stat-num">24/7</span><span class="stat-label">Available</span></div>
          <div class="stat-div"></div>
          <div class="stat"><span class="stat-num">COLRIS</span><span class="stat-label">Integrated</span></div>
          <div class="stat-div"></div>
          <div class="stat"><span class="stat-num">100%</span><span class="stat-label">Free for CU</span></div>
        </div>
      </div>
      <div class="hero-logo">
        <img src="/cu-logo.png" alt="Covenant University" class="hero-cu-logo" />
      </div>
    </section>

    <section class="features">
      <div class="section-label">WHAT IT CAN DO</div>
      <h2 class="section-title">Everything you need from the library, in one conversation</h2>
      <div class="features-grid">
        <div class="feature-card animate-card" v-for="(card, i) in featureCards" :key="i" :style="{ animationDelay: i * 0.1 + 's' }">
          <div class="feature-icon">{{ card.icon }}</div>
          <h3>{{ card.title }}</h3>
          <p>{{ card.desc }}</p>
        </div>
      </div>
    </section>

    <section class="how">
      <div class="section-label">HOW IT WORKS</div>
      <h2 class="section-title">Up and running in 3 steps</h2>
      <div class="steps">
        <div class="step">
          <div class="step-num">01</div>
          <h3>Register</h3>
          <p>Sign up with your Covenant University email address (@stu.cu.edu.ng or @covenantuniversity.edu.ng)</p>
        </div>
        <div class="step-arrow">→</div>
        <div class="step">
          <div class="step-num">02</div>
          <h3>Verify</h3>
          <p>Enter the 6-digit OTP sent to your institutional email to verify your identity</p>
        </div>
        <div class="step-arrow">→</div>
        <div class="step">
          <div class="step-num">03</div>
          <h3>Start Chatting</h3>
          <p>Ask anything about the library — books, hours, fines, databases, or general questions</p>
        </div>
      </div>
    </section>

    <section class="cta">
      <img src="/cu-logo.png" alt="CU" class="cta-logo" />
      <h2>Ready to transform how you use the Covenant University Library?</h2>
      <p>Join students already using COLRIS AI Assistant for smarter library access.</p>
      <button @click="$emit('show-register')" class="btn-primary large">Get Started Free</button>
    </section>

    <footer class="footer">
      <div class="footer-brand">
        <img src="/cu-logo.png" alt="CU" class="footer-logo" />
        <span>COLRIS AI Assistant · Covenant University Library</span>
      </div>
      <p class="footer-sub">Ota, Ogun State, Nigeria · Raising A New Generation Of Leaders</p>
      <p class="footer-copy">2026 Onuorah Ebubechukwu · Computer Science · Covenant University</p>
    </footer>
  </div>
</template>

<script>
export default {
  name: "Landing",
  emits: ["show-login", "show-register"],
  data() {
    return {
      typedText: "",
      fullText: "Your 24/7 AI Library",
      typingIndex: 0,
      featureCards: [
        { icon: "🔍", title: "Book Discovery", desc: "Search the COLRIS catalogue instantly. Get direct links to any book, journal or resource." },
        { icon: "📝", title: "Citation Generator", desc: "Generate APA, MLA, or Harvard citations in seconds. Just provide the book details." },
        { icon: "📥", title: "Book Requests", desc: "Cannot find a book? Submit a request directly to the librarian through the chat." },
        { icon: "🗄️", title: "Database Access", desc: "Get guided access to JSTOR, IEEE Xplore, Google Scholar and other academic databases." },
        { icon: "🧑‍💼", title: "Human Handoff", desc: "When the AI cannot help, it connects you directly to a librarian via email." },
        { icon: "📖", title: "Subject Guides", desc: "Tell the chatbot your topic and get a curated research guide with keywords and resources." }
      ]
    }
  },
  mounted() {
    this.startTyping()
    this.initParticles()
    this.initScrollAnimations()
  },
  methods: {
    startTyping() {
      if (this.typingIndex < this.fullText.length) {
        this.typedText += this.fullText[this.typingIndex]
        this.typingIndex++
        setTimeout(this.startTyping, 60)
      }
    },
    initParticles() {
      const canvas = this.$refs.particles
      if (!canvas) return
      const ctx = canvas.getContext("2d")
      canvas.width = window.innerWidth
      canvas.height = window.innerHeight
      const particles = []
      for (let i = 0; i < 60; i++) {
        particles.push({
          x: Math.random() * canvas.width,
          y: Math.random() * canvas.height,
          r: Math.random() * 1.5 + 0.5,
          dx: (Math.random() - 0.5) * 0.4,
          dy: (Math.random() - 0.5) * 0.4,
          opacity: Math.random() * 0.5 + 0.1
        })
      }
      const animate = () => {
        ctx.clearRect(0, 0, canvas.width, canvas.height)
        particles.forEach(p => {
          ctx.beginPath()
          ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
          ctx.fillStyle = `rgba(124, 58, 237, ${p.opacity})`
          ctx.fill()
          p.x += p.dx
          p.y += p.dy
          if (p.x < 0 || p.x > canvas.width) p.dx *= -1
          if (p.y < 0 || p.y > canvas.height) p.dy *= -1
        })
        requestAnimationFrame(animate)
      }
      animate()
      window.addEventListener("resize", () => {
        canvas.width = window.innerWidth
        canvas.height = window.innerHeight
      })
    },
    initScrollAnimations() {
      const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            entry.target.classList.add("visible")
          }
        })
      }, { threshold: 0.1 })
      setTimeout(() => {
        document.querySelectorAll(".animate-card, .step, .cta").forEach(el => observer.observe(el))
      }, 500)
    }
  }
}
</script>

<style scoped>
* { margin: 0; padding: 0; box-sizing: border-box; }
.landing { background: #0f0905; color: #f5f3ff; font-family: "Segoe UI", Arial, sans-serif; min-height: 100vh; overflow-x: hidden; }

/* Animated background */
.landing::before { content: ""; position: fixed; top: -50%; left: -50%; width: 200%; height: 200%; background: radial-gradient(ellipse at 20% 20%, rgba(124,58,237,0.06) 0%, transparent 50%), radial-gradient(ellipse at 80% 80%, rgba(26,15,10,0.8) 0%, transparent 50%), radial-gradient(ellipse at 50% 50%, rgba(92,61,46,0.04) 0%, transparent 70%); animation: bgPulse 8s ease-in-out infinite alternate; pointer-events: none; z-index: 0; }
@keyframes bgPulse { 0% { transform: scale(1) rotate(0deg); opacity: 0.8; } 100% { transform: scale(1.05) rotate(1deg); opacity: 1; } }
.landing > * { position: relative; z-index: 1; }
.nav { display: flex; justify-content: space-between; align-items: center; padding: 16px 60px; border-bottom: 1px solid rgba(124,58,237,0.15); position: sticky; top: 0; background: rgba(15,9,5,0.96); backdrop-filter: blur(10px); z-index: 10; }
.nav-brand { display: flex; align-items: center; gap: 12px; }
.cu-logo { width: 48px; height: 48px; object-fit: contain; }
.nav-brand-text { display: flex; flex-direction: column; }
.nav-title { font-size: 16px; font-weight: 800; color: #f5f3ff; line-height: 1.2; }
.nav-sub { font-size: 11px; color: #7c3aed; font-weight: 600; }
.nav-actions { display: flex; gap: 12px; }
.nav-login { background: transparent; border: 1.5px solid rgba(124,58,237,0.4); color: #7c3aed; padding: 8px 20px; border-radius: 8px; cursor: pointer; font-size: 14px; font-weight: 600; transition: all 0.2s; }
.nav-login:hover { background: rgba(124,58,237,0.1); }
.nav-register { background: #7c3aed; border: none; color: #0f0905; padding: 8px 20px; border-radius: 8px; cursor: pointer; font-size: 14px; font-weight: 700; transition: all 0.2s; }
.nav-register:hover { background: #e0c060; }
.hero { display: flex; align-items: center; justify-content: space-between; padding: 80px 60px; max-width: 1100px; margin: 0 auto; gap: 60px; }
.hero-inner { flex: 1; }
.hero-logo { flex-shrink: 0; }
.hero-cu-logo { width: 300px; height: 300px; object-fit: contain; opacity: 1; filter: drop-shadow(0 0 60px rgba(124,58,237,0.4)) drop-shadow(0 0 20px rgba(124,58,237,0.2)); animation: logoFloat 4s ease-in-out infinite; }
@keyframes logoFloat { 0%, 100% { transform: translateY(0px); filter: drop-shadow(0 0 60px rgba(124,58,237,0.4)); } 50% { transform: translateY(-12px); filter: drop-shadow(0 0 80px rgba(124,58,237,0.6)); } }
.hero-badge { display: inline-block; background: rgba(124,58,237,0.12); border: 1px solid rgba(124,58,237,0.35); color: #7c3aed; padding: 8px 18px; border-radius: 20px; font-size: 13px; font-weight: 600; margin-bottom: 28px; animation: badgePulse 3s ease-in-out infinite; }
@keyframes badgePulse { 0%, 100% { box-shadow: 0 0 0 0 rgba(124,58,237,0.2); } 50% { box-shadow: 0 0 0 6px rgba(124,58,237,0); } }
.hero-title { font-size: 56px; font-weight: 900; line-height: 1.1; margin-bottom: 24px; color: #f5f3ff; letter-spacing: -1px; }
.gold { color: #7c3aed; text-shadow: 0 0 40px rgba(124,58,237,0.4); }
.hero-sub { font-size: 16px; color: #b8a898; line-height: 1.7; margin-bottom: 40px; max-width: 500px; }
.hero-actions { display: flex; gap: 16px; margin-bottom: 48px; }
.btn-primary { background: linear-gradient(135deg, #7c3aed, #e0c060); color: #0f0905; border: none; padding: 14px 32px; border-radius: 10px; font-size: 15px; font-weight: 700; cursor: pointer; transition: all 0.2s; box-shadow: 0 4px 20px rgba(124,58,237,0.3); }
.btn-primary:hover { background: #e0c060; transform: translateY(-2px); }
.btn-primary.large { padding: 18px 40px; font-size: 17px; }
.btn-secondary { background: transparent; color: #f5f3ff; border: 1.5px solid rgba(253,246,227,0.3); padding: 14px 32px; border-radius: 10px; font-size: 15px; font-weight: 600; cursor: pointer; transition: all 0.2s; }
.btn-secondary:hover { border-color: #f5f3ff; }
.hero-stats { display: flex; align-items: center; gap: 32px; }
.stat { text-align: left; }
.stat-num { display: block; font-size: 28px; font-weight: 900; color: #7c3aed; text-shadow: 0 0 20px rgba(124,58,237,0.3); }
.stat-label { font-size: 11px; color: #7c3aed; text-transform: uppercase; letter-spacing: 0.5px; }
.stat-div { width: 1px; height: 36px; background: rgba(124,58,237,0.2); }
.features { padding: 80px 60px; max-width: 1100px; margin: 0 auto; position: relative; }
.section-label { font-size: 12px; font-weight: 700; color: #7c3aed; text-transform: uppercase; letter-spacing: 2px; margin-bottom: 16px; }
.section-title { font-size: 34px; font-weight: 800; color: #f5f3ff; margin-bottom: 48px; line-height: 1.2; }
.features-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; }
.feature-card { background: linear-gradient(135deg, rgba(255,255,255,0.04) 0%, rgba(124,58,237,0.02) 100%); border: 1px solid rgba(124,58,237,0.15); border-radius: 16px; padding: 28px; transition: all 0.3s; position: relative; overflow: hidden; }
.feature-card::before { content: ""; position: absolute; top: 0; left: 0; right: 0; height: 2px; background: linear-gradient(90deg, transparent, rgba(124,58,237,0.4), transparent); opacity: 0; transition: opacity 0.3s; }
.feature-card:hover::before { opacity: 1; }
.feature-card:hover { border-color: rgba(124,58,237,0.4); background: rgba(124,58,237,0.05); transform: translateY(-4px); }
.feature-icon { font-size: 28px; margin-bottom: 14px; }
.feature-card h3 { font-size: 16px; font-weight: 700; color: #f5f3ff; margin-bottom: 8px; }
.feature-card p { font-size: 13px; color: #7c3aed; line-height: 1.6; }
.how { padding: 80px 60px; background: linear-gradient(180deg, rgba(124,58,237,0.03) 0%, rgba(124,58,237,0.06) 50%, rgba(124,58,237,0.03) 100%); border-top: 1px solid rgba(124,58,237,0.15); border-bottom: 1px solid rgba(124,58,237,0.15); }
.how .section-label, .how .section-title { text-align: center; }
.steps { display: flex; align-items: center; justify-content: center; gap: 20px; max-width: 900px; margin: 0 auto; }
.step { background: rgba(255,255,255,0.03); border: 1px solid rgba(124,58,237,0.15); border-radius: 16px; padding: 32px 24px; flex: 1; }
.step-num { font-size: 36px; font-weight: 900; color: rgba(124,58,237,0.3); margin-bottom: 12px; }
.step h3 { font-size: 17px; font-weight: 700; color: #f5f3ff; margin-bottom: 8px; }
.step p { font-size: 13px; color: #7c3aed; line-height: 1.6; }
.step-arrow { font-size: 24px; color: #7c3aed; flex-shrink: 0; }
.cta { text-align: center; padding: 100px 60px; }
.cta-logo { width: 80px; height: 80px; object-fit: contain; margin-bottom: 24px; opacity: 0.8; }
.cta h2 { font-size: 34px; font-weight: 800; color: #f5f3ff; margin-bottom: 16px; line-height: 1.2; }
.cta p { font-size: 15px; color: #7c3aed; margin-bottom: 36px; }
.footer { border-top: 1px solid rgba(124,58,237,0.15); padding: 40px 60px; text-align: center; }
.footer-brand { display: flex; align-items: center; justify-content: center; gap: 10px; font-size: 15px; font-weight: 700; color: #f5f3ff; margin-bottom: 10px; }
.footer-logo { width: 32px; height: 32px; object-fit: contain; }
.footer-sub { font-size: 13px; color: #7c3aed; margin-bottom: 6px; }
.footer-copy { font-size: 12px; color: #5c4a3a; }
.particles-canvas { position: fixed; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 0; }
.cursor { animation: blink 1s step-end infinite; color: #7c3aed; }
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }
.animate-card { opacity: 0; transform: translateY(30px); transition: opacity 0.6s ease, transform 0.6s ease; }
.animate-card.visible { opacity: 1; transform: translateY(0); }
.step { opacity: 0; transform: translateY(20px); transition: opacity 0.6s ease, transform 0.6s ease; }
.step.visible { opacity: 1; transform: translateY(0); }
.cta { opacity: 0; transition: opacity 0.8s ease; }
.cta.visible { opacity: 1; }
@media (max-width: 768px) {
  .nav { padding: 16px 20px; }
  .hero { flex-direction: column; padding: 40px 20px; text-align: center; }
  .hero-logo { display: none; }
  .hero-title { font-size: 36px; }
  .hero-actions { justify-content: center; }
  .hero-stats { justify-content: center; }
  .features { padding: 60px 20px; }
  .features-grid { grid-template-columns: 1fr; }
  .how { padding: 60px 20px; }
  .steps { flex-direction: column; }
  .step-arrow { transform: rotate(90deg); }
  .cta { padding: 60px 20px; }
  .footer { padding: 40px 20px; }
}
</style>