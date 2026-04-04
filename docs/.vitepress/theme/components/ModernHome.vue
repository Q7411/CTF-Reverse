<script setup>
import { onMounted, ref, onUnmounted, watch } from 'vue'
import { useData } from 'vitepress'
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

const { isDark } = useData()

const heroRef = ref(null)
const bgCanvas = ref(null)
const contentRef = ref(null)
const skillsRef = ref(null)
const statsRef = ref(null)
const postsRef = ref(null)
let ctx
let canvasCtx
let animationFrameId
let particles = []
const mouse = { x: null, y: null, radius: 150 }

class Particle {
  constructor(canvas) {
    this.canvas = canvas
    this.x = Math.random() * canvas.width
    this.y = Math.random() * canvas.height
    this.size = Math.random() * 2 + 1
    this.baseX = this.x
    this.baseY = this.y
    this.density = (Math.random() * 20) + 1
    this.vx = (Math.random() - 0.5) * 0.8
    this.vy = (Math.random() - 0.5) * 0.8
  }

  draw() {
    if (!canvasCtx) return
    canvasCtx.fillStyle = isDark.value ? 'rgba(0, 229, 255, 0.8)' : 'rgba(10, 10, 11, 0.4)'
    canvasCtx.beginPath()
    canvasCtx.arc(this.x, this.y, this.size, 0, Math.PI * 2)
    canvasCtx.closePath()
    canvasCtx.fill()
  }

  update() {
    this.x += this.vx
    this.y += this.vy

    // Bounce
    if (this.x < 0 || this.x > this.canvas.width) this.vx *= -1
    if (this.y < 0 || this.y > this.canvas.height) this.vy *= -1

    // Mouse interaction
    if (mouse.x != null && mouse.y != null) {
      let dx = mouse.x - this.x
      let dy = mouse.y - this.y
      let distance = Math.sqrt(dx * dx + dy * dy)
      
      if (distance < mouse.radius) {
        const forceDirectionX = dx / distance
        const forceDirectionY = dy / distance
        const force = (mouse.radius - distance) / mouse.radius
        const directionX = forceDirectionX * force * this.density
        const directionY = forceDirectionY * force * this.density
        
        this.x -= directionX
        this.y -= directionY
      }
    }
    this.draw()
  }
}

const initParticles = () => {
  particles = []
  if (!bgCanvas.value) return
  const numberOfParticles = window.innerWidth < 768 ? 50 : 120
  for (let i = 0; i < numberOfParticles; i++) {
    particles.push(new Particle(bgCanvas.value))
  }
}

const animate = () => {
  if (!canvasCtx || !bgCanvas.value) return
  canvasCtx.clearRect(0, 0, bgCanvas.value.width, bgCanvas.value.height)
  
  for (let i = 0; i < particles.length; i++) {
    particles[i].update()
    
    // Connect particles
    for (let j = i; j < particles.length; j++) {
      const dx = particles[i].x - particles[j].x
      const dy = particles[i].y - particles[j].y
      const distance = Math.sqrt(dx * dx + dy * dy)
      
      if (distance < 150) {
        canvasCtx.beginPath()
        const opacity = 1 - (distance / 150)
        canvasCtx.strokeStyle = isDark.value 
          ? `rgba(0, 229, 255, ${opacity * 0.3})` 
          : `rgba(10, 10, 11, ${opacity * 0.15})`
        canvasCtx.lineWidth = 1
        canvasCtx.moveTo(particles[i].x, particles[i].y)
        canvasCtx.lineTo(particles[j].x, particles[j].y)
        canvasCtx.stroke()
        canvasCtx.closePath()
      }
    }
  }
  animationFrameId = requestAnimationFrame(animate)
}

const handleResize = () => {
  if (!bgCanvas.value) return
  bgCanvas.value.width = window.innerWidth
  bgCanvas.value.height = window.innerHeight
  initParticles()
}

const handleMouseMove = (event) => {
  if (!heroRef.value) return
  const rect = heroRef.value.getBoundingClientRect()
  mouse.x = event.clientX - rect.left
  mouse.y = event.clientY - rect.top
}

const handleMouseOut = () => {
  mouse.x = null
  mouse.y = null
}

const displayTitle1 = ref('欢迎来到我的博客')
const displayTitle2 = ref('HELLO WORLD_')
const displaySubtitle = ref('专注于 CTF 逆向工程、安全研究与技术分享，记录每一次在二进制世界中的探索与成长。')
const cursor1 = ref(false)
const cursor2 = ref(false)
const cursor3 = ref(false)

const typeWriter = (text, refVar, cursorVar, speed = 50) => {
  return new Promise((resolve) => {
    cursorVar.value = true
    let i = 0
    refVar.value = ''
    const timer = setInterval(() => {
      if (i < text.length) {
        refVar.value += text.charAt(i)
        i++
      } else {
        clearInterval(timer)
        cursorVar.value = false
        resolve()
      }
    }, speed)
  })
}

onMounted(() => {
  // 性能降级：移动端或偏好减少动画时不渲染粒子
  if (!window.matchMedia('(max-width: 768px)').matches && !window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
    if (bgCanvas.value) {
      canvasCtx = bgCanvas.value.getContext('2d')
      bgCanvas.value.width = window.innerWidth
      bgCanvas.value.height = window.innerHeight
      
      initParticles()
      animate()

      window.addEventListener('resize', handleResize)
      if (heroRef.value) {
        heroRef.value.addEventListener('mousemove', handleMouseMove)
        heroRef.value.addEventListener('mouseout', handleMouseOut)
      }
    }
  }

  // GSAP 视差动画
  ctx = gsap.context(() => {
    // 英雄区 Canvas 视差滚动
    gsap.to('.hero-canvas', {
      yPercent: 30,
      ease: 'none',
      scrollTrigger: {
        trigger: heroRef.value,
        start: 'top top',
        end: 'bottom top',
        scrub: true
      }
    })

    // 标题入场动画
    gsap.fromTo('.hero-title', 
      { opacity: 0, y: 50 },
      { opacity: 1, y: 0, duration: 1.2, ease: 'power4.out', delay: 0.2 }
    )
    gsap.fromTo('.hero-subtitle', 
      { opacity: 0, y: 30 },
      { opacity: 1, y: 0, duration: 1, ease: 'power3.out', delay: 0.5 }
    )
    gsap.fromTo('.hero-actions', 
      { opacity: 0, y: 20 },
      { opacity: 1, y: 0, duration: 0.8, ease: 'power2.out', delay: 0.8 }
    )



    // 技能标签入场动画
    gsap.fromTo('.skill-tag',
      { opacity: 0, scale: 0.8, y: 20 },
      {
        opacity: 1,
        scale: 1,
        y: 0,
        duration: 0.6,
        stagger: 0.05,
        ease: 'back.out(1.5)',
        scrollTrigger: {
          trigger: skillsRef.value,
          start: 'top 80%',
          toggleActions: 'play none none reverse'
        }
      }
    )

    // 最新文章卡片入场动画
    gsap.fromTo('.post-card',
      { opacity: 0, x: -50 },
      {
        opacity: 1,
        x: 0,
        duration: 0.8,
        stagger: 0.2,
        ease: 'power3.out',
        scrollTrigger: {
          trigger: postsRef.value,
          start: 'top 80%',
          toggleActions: 'play none none reverse'
        }
      }
    )

    // 数据统计动画
    gsap.fromTo('.stat-item',
      { opacity: 0, y: 30 },
      {
        opacity: 1,
        y: 0,
        duration: 0.8,
        stagger: 0.1,
        ease: 'power3.out',
        scrollTrigger: {
          trigger: statsRef.value,
          start: 'top 85%',
          toggleActions: 'play none none reverse'
        }
      }
    )

    // 动态打字机效果
    displayTitle1.value = '\u200B'
    displayTitle2.value = '\u200B'
    displaySubtitle.value = '\u200B'

    setTimeout(async () => {
      await typeWriter('欢迎来到我的博客', displayTitle1, cursor1, 100)
      await typeWriter('HELLO WORLD_', displayTitle2, cursor2, 80)
      await typeWriter('专注于 CTF 逆向工程、安全研究与技术分享，记录每一次在二进制世界中的探索与成长。', displaySubtitle, cursor3, 40)
      cursor3.value = true // 保持最后一个光标闪烁
    }, 800)

  })
})

onUnmounted(() => {
  if (ctx) ctx.revert()
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId)
  }
  window.removeEventListener('resize', handleResize)
  if (heroRef.value) {
    heroRef.value.removeEventListener('mousemove', handleMouseMove)
    heroRef.value.removeEventListener('mouseout', handleMouseOut)
  }
})



const skills = [
  '逆向工程 (Reverse)', '二进制安全 (Pwn)', 'Web 安全渗透', '密码学 (Crypto)',
  '汇编语言 (Assembly)', 'Python', 'C / C++', 'Vue 3 & Vite', 'GSAP 动效', 'UI/UX 设计'
]

const recentPosts = [
  {
    title: 'BASE64 加密原理与逆向',
    excerpt: '计算机以 8 位（bit）为一个字节，而 Base64 以 6 位为一个单元，探索其变表、魔改运算及爆破细节...',
    date: '2024-04-02',
    link: '/CTF-Reverse/BASE64',
    category: 'CTF-Reverse'
  },
  {
    title: '我的第一篇博客文章',
    excerpt: '记录技术成长，分享开发经验与安全研究的心得体会...',
    date: '2024-04-01',
    link: '/posts/first-post',
    category: '随笔'
  }
]
</script>

<template>
  <div class="modern-home">
    <!-- Cyber Background Elements -->
    <div class="cyber-grid"></div>
    <div class="ambient-orb orb-1"></div>
    <div class="ambient-orb orb-2"></div>

    <!-- Hero Section -->
    <section ref="heroRef" class="hero-section">
      <div class="video-container">
        <canvas ref="bgCanvas" class="hero-canvas gpu-accel"></canvas>
        <div class="video-overlay"></div>
      </div>

      <!-- HUD Elements -->
      <div class="hud-top-left">SYS.INIT // {{ new Date().getFullYear() }}.CTF.REV</div>
      <div class="hud-bottom-right">STATUS: <span class="status-online">ONLINE</span> <span class="blink">_</span></div>

      <div ref="contentRef" class="hero-content">
        <div class="terminal-badge">
          <span class="terminal-dot red"></span>
          <span class="terminal-dot yellow"></span>
          <span class="terminal-dot green"></span>
          <span class="terminal-text">root@q7411:~# ./start_journey.sh</span>
        </div>
        <h1 class="hero-title gpu-accel">
          <span class="liquid-text" :data-text="displayTitle1 === '\u200B' ? '' : displayTitle1">{{ displayTitle1 }}</span><span v-show="cursor1" class="typing-cursor">_</span><br/>
          <span class="cyber-text" :data-text="displayTitle2 === '\u200B' ? '' : displayTitle2">{{ displayTitle2 }}</span><span v-show="cursor2" class="typing-cursor">_</span>
        </h1>
        <p class="hero-subtitle gpu-accel">
          {{ displaySubtitle }}<span v-show="cursor3" class="typing-cursor">_</span>
        </p>
        <div class="hero-actions gpu-accel">
          <a href="/posts/first-post" class="btn btn-primary cyber-btn">
            <span class="btn-content">开始探索</span>
            <span class="btn-glitch"></span>
          </a>
          <a href="/about" class="btn btn-secondary">关于我们</a>
        </div>
      </div>
    </section>



    <!-- Skills Section -->
    <section ref="skillsRef" class="skills-section">
      <h2 class="section-title">Core Skills</h2>
      <div class="skills-container">
        <span 
          v-for="(skill, index) in skills" 
          :key="index"
          class="skill-tag gpu-accel"
        >
          {{ skill }}
        </span>
      </div>
    </section>

    <!-- Stats Section -->
    <section ref="statsRef" class="stats-section">
      <div class="stats-grid">
        <div class="stat-item gpu-accel">
          <div class="stat-number">0xff</div>
          <div class="stat-label">BINARIES PWNED</div>
        </div>
        <div class="stat-item gpu-accel">
          <div class="stat-number">10K+</div>
          <div class="stat-label">LINES OF CODE</div>
        </div>
        <div class="stat-item gpu-accel">
          <div class="stat-number">24/7</div>
          <div class="stat-label">SYSTEM UPTIME</div>
        </div>
        <div class="stat-item gpu-accel">
          <div class="stat-number">0.05</div>
          <div class="stat-label">CLS SCORE</div>
        </div>
      </div>
    </section>

    <!-- Recent Posts Section -->
    <section ref="postsRef" class="posts-section">
      <h2 class="section-title">Recent Discoveries</h2>
      <div class="posts-grid">
        <a 
          v-for="(post, index) in recentPosts" 
          :key="index" 
          :href="post.link" 
          class="post-card glass-card gpu-accel"
        >
          <div class="post-meta">
            <span class="post-category">{{ post.category }}</span>
            <span class="post-date">{{ post.date }}</span>
          </div>
          <h3 class="post-title">{{ post.title }}</h3>
          <p class="post-excerpt">{{ post.excerpt }}</p>
          <div class="post-read-more">阅读全文 ➔</div>
        </a>
      </div>
    </section>


  </div>
</template>

<style scoped>
.modern-home {
  width: 100%;
  background-color: var(--vp-c-bg); /* Use theme background */
  color: var(--vp-c-text-1);
  overflow: hidden;
  transition: background-color 0.5s ease;
}

/* Hero Section */
.hero-section {
  position: relative;
  height: 100vh;
  min-height: 800px;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  border-bottom: 1px solid rgba(0, 229, 255, 0.1);
}

/* Cyber Background Elements */
.cyber-grid {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background-image: 
    linear-gradient(var(--vp-c-divider) 1px, transparent 1px),
    linear-gradient(90deg, var(--vp-c-divider) 1px, transparent 1px);
  background-size: 50px 50px;
  z-index: 1;
  pointer-events: none;
  mask-image: linear-gradient(to bottom, rgba(0,0,0,1) 0%, rgba(0,0,0,0) 100%);
  -webkit-mask-image: linear-gradient(to bottom, rgba(0,0,0,1) 0%, rgba(0,0,0,0) 100%);
  opacity: 0.5;
}

.ambient-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(100px);
  opacity: 0.3;
  z-index: 0;
  pointer-events: none;
  animation: floatOrb 12s infinite alternate ease-in-out;
}
.orb-1 { top: -10%; left: -10%; width: 40vw; height: 40vw; background: var(--vp-c-brand-1); }
.orb-2 { bottom: 20%; right: -10%; width: 50vw; height: 50vw; background: var(--vp-c-cyan-1); animation-delay: -6s; }

@keyframes floatOrb {
  0% { transform: translate(0, 0) scale(1); }
  100% { transform: translate(50px, 80px) scale(1.1); }
}

/* HUD Elements */
.hud-top-left {
  position: absolute;
  top: 100px;
  left: 40px;
  font-family: var(--vp-font-family-mono);
  font-size: 0.85rem;
  color: var(--vp-c-brand-1);
  letter-spacing: 0.2em;
  z-index: 10;
  opacity: 0.7;
}

.hud-top-left::before {
  content: '';
  display: inline-block;
  width: 12px; height: 12px;
  background: var(--vp-c-brand-1);
  margin-right: 12px;
  animation: pulse 2s infinite;
}

.hud-bottom-right {
  position: absolute;
  bottom: 40px;
  right: 40px;
  font-family: var(--vp-font-family-mono);
  font-size: 0.85rem;
  color: var(--vp-c-text-3);
  letter-spacing: 0.1em;
  z-index: 10;
}

.status-online {
  color: var(--vp-c-cyan-1);
  text-shadow: 0 0 10px var(--vp-c-cyan-soft);
}

.typing-cursor {
  display: inline-block;
  color: var(--vp-c-brand-1);
  animation: blink 1s step-end infinite;
  font-weight: bold;
}

.blink {
  animation: blink 1s step-end infinite;
}

@keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }
@keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }

/* Terminal Badge */
.terminal-badge {
  display: inline-flex;
  align-items: center;
  background: var(--vp-c-bg-soft);
  backdrop-filter: blur(10px);
  border: 1px solid var(--vp-c-divider);
  padding: 8px 20px;
  border-radius: 8px;
  margin-bottom: 30px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.terminal-dot {
  width: 10px; height: 10px; border-radius: 50%; margin-right: 8px;
}
.terminal-dot.red { background: #ff5f56; }
.terminal-dot.yellow { background: #ffbd2e; }
.terminal-dot.green { background: #27c93f; margin-right: 16px; }

.terminal-text {
  font-family: var(--vp-font-family-mono);
  font-size: 0.9rem;
  color: var(--vp-c-text-2);
}

.video-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
}

.hero-canvas {
  width: 100%;
  height: 120%; /* 留出视差滚动空间 */
  position: absolute;
  top: -10%;
  left: 0;
  pointer-events: none;
}

.video-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    to bottom,
    var(--vp-c-bg-soft) 0%,
    var(--vp-c-bg) 100%
  );
  opacity: 0.6;
  z-index: 1;
  pointer-events: none;
}

.hero-content {
  position: relative;
  z-index: 2;
  text-align: center;
  max-width: 900px;
  padding: 0 24px;
}

.hero-title {
  font-family: var(--vp-font-family-base);
  font-size: clamp(3.5rem, 8vw, 6rem);
  font-weight: 800;
  line-height: 1.3;
  letter-spacing: -0.04em;
  margin-bottom: 24px;
  color: var(--vp-c-text-1);
  padding: 10px 0;
}

.liquid-text {
  position: relative;
  display: inline-block;
  color: transparent;
  -webkit-text-stroke: 2px var(--vp-c-text-2); /* 镂空文字的描边 */
  padding-bottom: 0.1em;
  z-index: 1;
}

/* 基础水波层配置：利用多重背景叠加和动画实现不规则真实海浪 */
.liquid-text::before {
  content: attr(data-text);
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  -webkit-text-stroke: 0px transparent;
  color: transparent;
  -webkit-background-clip: text;
  background-clip: text;
  
  /* 多重 SVG 海浪背景层叠 */
  background-image: 
    /* 前浪 - 亮青色，高频小浪 */
    url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1000 1000" preserveAspectRatio="none"><path d="M0,1000 L0,250 Q150,150 250,250 T500,250 T750,250 T1000,250 L1000,1000 Z" fill="%2300e5ff" opacity="0.8"/></svg>'),
    /* 中浪 - 品牌蓝，中频中浪 */
    url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1000 1000" preserveAspectRatio="none"><path d="M0,1000 L0,280 Q200,380 350,280 T700,280 T1000,280 L1000,1000 Z" fill="%233498db" opacity="0.6"/></svg>'),
    /* 底浪 - 深海蓝，低频大浪 */
    url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1000 1000" preserveAspectRatio="none"><path d="M0,1000 L0,300 Q250,100 500,300 T1000,300 L1000,1000 Z" fill="%231a5276" opacity="0.9"/></svg>');
  
  /* 分别设置三个海浪的尺寸：使其周期不同，打破规律感 */
  background-size: 50% 100%, 75% 100%, 120% 100%;
  background-repeat: repeat-x, repeat-x, repeat-x;
  
  /* 让文字本身成为这三层海浪的蒙版 */
  animation: oceanSurge 8s cubic-bezier(0.36, 0.45, 0.63, 0.53) infinite alternate,
             oceanFlow 12s linear infinite;
  z-index: 2;
}

/* 横向流动（不同层速度不同） */
@keyframes oceanFlow {
  0% {
    background-position: 0% 100%, 0% 100%, 0% 100%;
  }
  100% {
    background-position: -100% 100%, -150% 100%, 50% 100%;
  }
}

/* 纵向非规则翻涌（模拟潮汐的上下拍打、挤压变形） */
@keyframes oceanSurge {
  0% {
    background-size: 50% 90%, 75% 95%, 120% 100%;
  }
  25% {
    background-size: 55% 110%, 80% 90%, 110% 105%;
  }
  50% {
    background-size: 60% 85%, 70% 115%, 130% 95%;
  }
  75% {
    background-size: 45% 105%, 85% 85%, 115% 110%;
  }
  100% {
    background-size: 50% 100%, 75% 100%, 120% 90%;
  }
}

.cyber-text {
  font-family: var(--vp-font-family-mono);
  font-size: clamp(1.5rem, 4vw, 3rem);
  color: var(--vp-c-cyan-1);
  text-shadow: 0 0 20px var(--vp-c-cyan-soft);
  position: relative;
  display: inline-block;
  letter-spacing: 0.1em;
  padding-bottom: 0.1em;
}

.hero-subtitle {
  font-size: clamp(1.2rem, 3vw, 1.5rem);
  color: var(--vp-c-text-2);
  max-width: 600px;
  margin: 0 auto 48px;
  line-height: 1.6;
}

.hero-actions {
  display: flex;
  gap: 20px;
  justify-content: center;
  flex-wrap: wrap;
}

.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 16px 40px;
  border-radius: 100px;
  font-weight: 600;
  font-size: 1.1rem;
  text-decoration: none;
  transition: all 0.4s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.btn-primary {
  background-color: var(--vp-c-brand-1);
  color: #0A0A0B; /* Text always dark on brand button */
  box-shadow: 0 0 20px rgba(212, 175, 55, 0.4);
}

.btn-primary:hover {
  background-color: var(--vp-c-brand-2);
  transform: translateY(-4px) scale(1.02);
  box-shadow: 0 10px 30px rgba(212, 175, 55, 0.6);
}

.btn-secondary {
  background-color: var(--vp-c-bg-soft);
  color: var(--vp-c-text-1);
  backdrop-filter: blur(10px);
  border: 1px solid var(--vp-c-divider);
}

.btn-secondary:hover {
  background-color: var(--vp-c-bg-mute);
  transform: translateY(-4px);
}



/* Section Title */
.section-title {
  text-align: center;
  font-size: clamp(2rem, 5vw, 3rem);
  font-weight: 800;
  margin-bottom: 60px;
  color: var(--vp-c-text-1);
  letter-spacing: -0.02em;
  background: linear-gradient(135deg, var(--vp-c-text-1) 50%, var(--vp-c-text-3));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  line-height: 1.3;
  padding-bottom: 0.1em;
}

/* Skills Section */
.skills-section {
  padding: 80px 24px;
  max-width: 1000px;
  margin: 0 auto;
}

.skills-container {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  justify-content: center;
}

.skill-tag {
  background: var(--vp-c-bg-soft);
  border: 1px solid var(--vp-c-divider);
  padding: 12px 24px;
  border-radius: 100px;
  font-size: 1rem;
  font-weight: 600;
  color: var(--vp-c-text-1);
  transition: all 0.3s ease;
  cursor: default;
}

.skill-tag:hover {
  background: var(--vp-c-brand-1);
  color: #0A0A0B;
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 10px 20px rgba(212, 175, 55, 0.3);
}

/* Stats Section */
.stats-section {
  padding: 40px 24px;
  max-width: 1000px;
  margin: 0 auto;
  border-top: 1px dashed var(--vp-c-divider);
  border-bottom: 1px dashed var(--vp-c-divider);
}

.stats-grid {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  gap: 40px;
  text-align: center;
}

.stat-item {
  flex: 1;
  min-width: 150px;
}

.stat-number {
  font-family: var(--vp-font-family-mono);
  font-size: 3rem;
  font-weight: 800;
  color: var(--vp-c-brand-1);
  margin-bottom: 8px;
  text-shadow: 0 0 15px rgba(212, 175, 55, 0.4);
}

.stat-label {
  font-size: 0.85rem;
  letter-spacing: 0.15em;
  color: var(--vp-c-text-2);
  font-family: var(--vp-font-family-mono);
}

/* Recent Posts Section */
.posts-section {
  padding: 80px 24px;
  max-width: 1280px;
  margin: 0 auto;
}

.posts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 32px;
}

.post-card {
  display: block;
  text-decoration: none;
  padding: 40px 32px;
  border-radius: 20px;
  transition: all 0.5s cubic-bezier(0.2, 0.8, 0.2, 1);
  background: var(--vp-c-bg-soft) !important;
  border: 1px solid var(--vp-c-divider) !important;
  position: relative;
  overflow: hidden;
}

.post-card::before {
  content: '';
  position: absolute;
  top: 0; left: 0;
  width: 4px; height: 0%;
  background: var(--vp-c-cyan-1);
  transition: height 0.4s ease;
}

.post-card:hover::before {
  height: 100%;
  box-shadow: 0 0 20px var(--vp-c-cyan-1);
}

.post-card:hover {
  transform: translateY(-8px) !important;
  border-color: var(--vp-c-brand-1) !important;
  box-shadow: 0 15px 30px rgba(0, 0, 0, 0.1) !important;
}

.post-meta {
  display: flex;
  justify-content: space-between;
  font-size: 0.85rem;
  margin-bottom: 16px;
  color: var(--vp-c-text-3);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.post-category {
  color: var(--vp-c-brand-1);
  font-weight: 600;
}

.post-title {
  font-size: 1.4rem;
  font-weight: 700;
  color: var(--vp-c-text-1);
  margin-bottom: 12px;
  line-height: 1.4;
  transition: color 0.3s ease;
}

.post-card:hover .post-title {
  color: var(--vp-c-brand-1);
}

.post-excerpt {
  font-size: 1rem;
  color: var(--vp-c-text-2);
  line-height: 1.6;
  margin-bottom: 24px;
}

.post-read-more {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--vp-c-brand-1);
  display: flex;
  align-items: center;
  gap: 8px;
}


/* Responsive */
@media (max-width: 768px) {
  .features-section {
    padding: 60px 24px;
    margin-top: 0;
  }
  .btn {
    width: 100%;
  }
}
</style>