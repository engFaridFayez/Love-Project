<script setup>
import { ref, reactive, onMounted, onBeforeUnmount } from 'vue'

/* ============ EDIT THIS PART ONLY ============ */
const C = {
  name: 'Merola',
  nick: 'Lolla',
  from: 'Your Name',                       // <- your name
  music: '/music.mp3',                     // put your song in /public/music.mp3
  heroLine: 'Every day with you feels like a birthday, Lolla.',
  letter: [
    'My Lolla,',
    'Today the world got the best thing it ever made. Thank you for every laugh, every late night, and every time you made a bad day feel small.',
    'I made this little corner of the internet so you can come back to it whenever you want to remember how loved you are.',
    'Happy birthday, my love. I choose you today, tomorrow, and always.'
  ]
}

// Put your photos in /public/images/ (1.jpg, 2.jpg ...) and write your memories
const memories = [
  { img: '/images/1.jpg', title: 'The day we met',   text: 'I did not know yet, but my life had just changed.', date: 'Write the date' },
  { img: '/images/2.jpg', title: 'Our first trip',   text: 'You laughed so hard that day. I still hear it.',    date: 'Write the date' },
  { img: '/images/3.jpg', title: 'That perfect night', text: 'Nothing special happened. It was perfect anyway.', date: 'Write the date' },
  { img: '/images/4.jpg', title: 'Just us',          text: 'My favorite place is wherever you are.',            date: 'Write the date' }
]

const quotes = [
  'You are my today and all of my tomorrows.',
  'In a sea of people, my eyes will always search for you.',
  'Home is not a place. It is you.',
  'I love you more than yesterday, less than tomorrow.',
  'With you, even silence feels like my favorite song.',
  'You are the reason I believe in beautiful things.'
]

const reasons = [
  { q: 'Your smile',      a: 'It fixes my whole day in one second.' },
  { q: 'Your kindness',   a: 'You make everyone around you feel safe.' },
  { q: 'Your laugh',      a: 'My favorite sound in the world.' },
  { q: 'Your strength',   a: 'I am proud of you every single day.' },
  { q: 'Just you',        a: 'No reason needed. You are Lolla.' },
  { q: 'Our future',      a: 'I cannot wait to see all of it with you.' }
]
/* ============================================= */

const opened = ref(false)
const playing = ref(false)
const typed = ref('')
const qi = ref(0)
const blown = ref(false)
const bg = ref(null)
const fx = ref(null)
let audio, raf, qTimer, io
let W = 0, H = 0, stars = [], hearts = [], parts = []

const vReveal = {
  mounted(el) { el.classList.add('rv'); io && io.observe(el) }
}

function toggleMusic() {
  if (!audio) return
  if (audio.paused) { audio.play(); playing.value = true }
  else { audio.pause(); playing.value = false }
}

function openGift(e) {
  audio = new Audio(C.music); audio.loop = true; audio.volume = 0.8
  audio.play().then(() => (playing.value = true)).catch(() => {})
  burst(e.clientX, e.clientY, 90, 14)
  setTimeout(() => (opened.value = true), 700)
  setTimeout(typeIt, 2600)
  qTimer = setInterval(() => (qi.value = (qi.value + 1) % quotes.length), 5200)
}

function typeIt() {
  let i = 0
  const t = setInterval(() => {
    typed.value = C.heroLine.slice(0, ++i)
    if (i >= C.heroLine.length) clearInterval(t)
  }, 55)
}

function blow(e) {
  if (blown.value) return
  blown.value = true
  const r = e.currentTarget.getBoundingClientRect()
  burst(r.left + r.width / 2, r.top + 40, 160, 18)
  let n = 0
  const t = setInterval(() => {
    burst(Math.random() * W, Math.random() * H * 0.6, 70, 12)
    if (++n > 14) clearInterval(t)
  }, 380)
}

/* ---------- particles ---------- */
const palette = ['#ff5c8a', '#ffd9a0', '#b79cff', '#ff9ec0', '#fff3d6', '#7fe0d0']
function burst(x, y, n, power = 10) {
  for (let i = 0; i < n; i++) {
    const a = Math.random() * Math.PI * 2, s = Math.random() * power + 2
    parts.push({
      x, y, vx: Math.cos(a) * s, vy: Math.sin(a) * s - 3, g: 0.2,
      life: 0, max: 90 + Math.random() * 70, size: 8 + Math.random() * 14,
      c: palette[(Math.random() * palette.length) | 0],
      heart: Math.random() > 0.45, rot: Math.random() * 6, vr: (Math.random() - 0.5) * 0.3
    })
  }
}
function trail(x, y) {
  parts.push({ x, y, vx: (Math.random() - .5) * 1.2, vy: -Math.random() * 1.5 - .4, g: -0.01,
    life: 0, max: 50, size: 10 + Math.random() * 8, c: palette[(Math.random() * 3) | 0], heart: true, rot: 0, vr: 0 })
}
const onMove = (e) => {
  if (!opened.value) return
  const p = e.touches ? e.touches[0] : e
  if (Math.random() > 0.45) trail(p.clientX, p.clientY)
}

function resize() {
  W = innerWidth; H = innerHeight
  for (const c of [bg.value, fx.value]) { c.width = W; c.height = H }
  stars = Array.from({ length: Math.min(160, W / 7) }, () => ({
    x: Math.random() * W, y: Math.random() * H, r: Math.random() * 1.4 + .3, p: Math.random() * 6
  }))
  hearts = Array.from({ length: W < 600 ? 14 : 26 }, () => newHeart(true))
}
function newHeart(anywhere) {
  return { x: Math.random() * W, y: anywhere ? Math.random() * H : H + 40,
    s: 10 + Math.random() * 26, v: .25 + Math.random() * .6, w: Math.random() * 6, a: .08 + Math.random() * .22 }
}

function loop(t) {
  const b = bg.value.getContext('2d'), f = fx.value.getContext('2d')
  b.clearRect(0, 0, W, H); f.clearRect(0, 0, W, H)
  for (const s of stars) {
    b.globalAlpha = .35 + Math.sin(t / 700 + s.p) * .35
    b.fillStyle = '#fff3d6'; b.beginPath(); b.arc(s.x, s.y, s.r, 0, 7); b.fill()
  }
  b.fillStyle = '#ff7aa2'
  for (const h of hearts) {
    h.y -= h.v; h.x += Math.sin(t / 1500 + h.w) * .4
    if (h.y < -40) Object.assign(h, newHeart(false))
    b.globalAlpha = h.a; b.font = h.s + 'px serif'; b.fillText('♥', h.x, h.y)
  }
  for (let i = parts.length - 1; i >= 0; i--) {
    const p = parts[i]
    p.life++; p.vy += p.g; p.vx *= .985; p.x += p.vx; p.y += p.vy; p.rot += p.vr
    if (p.life > p.max) { parts.splice(i, 1); continue }
    f.globalAlpha = 1 - p.life / p.max; f.fillStyle = p.c
    if (p.heart) { f.font = p.size + 'px serif'; f.fillText('♥', p.x, p.y) }
    else { f.save(); f.translate(p.x, p.y); f.rotate(p.rot); f.fillRect(-p.size / 4, -p.size / 8, p.size / 2, p.size / 4); f.restore() }
  }
  raf = requestAnimationFrame(loop)
}

onMounted(() => {
  io = new IntersectionObserver((es) => es.forEach((e) => {
    if (e.isIntersecting) { e.target.classList.add('in'); io.unobserve(e.target) }
  }), { threshold: 0.18 })
  resize(); addEventListener('resize', resize)
  addEventListener('mousemove', onMove); addEventListener('touchmove', onMove, { passive: true })
  raf = requestAnimationFrame(loop)
})
onBeforeUnmount(() => {
  cancelAnimationFrame(raf); clearInterval(qTimer); io && io.disconnect()
  removeEventListener('resize', resize); removeEventListener('mousemove', onMove); removeEventListener('touchmove', onMove)
  audio && audio.pause()
})
</script>

<template>
  <div class="app">
    <div class="aurora"><i></i><i></i><i></i></div>
    <canvas ref="bg" class="cv"></canvas>

    <!-- GATE -->
    <transition name="gate">
      <div v-if="!opened" class="gate">
        <p class="gate-hint">Something is waiting for you, {{ C.nick }}</p>
        <button class="env" @click="openGift" aria-label="Open your gift">
          <span class="env-flap"></span>
          <span class="env-seal">♥</span>
        </button>
        <p class="gate-tap">tap the heart</p>
      </div>
    </transition>

    <main v-if="opened">
      <button class="music" :class="{ on: playing }" @click="toggleMusic" :aria-label="playing ? 'Pause music' : 'Play music'">
        <span></span><span></span><span></span><span></span>
      </button>

      <!-- HERO -->
      <section class="hero">
        <h1 class="hb" aria-label="Happy Birthday">
          <span v-for="(ch, i) in 'Happy Birthday'" :key="i" :style="{ '--i': i }">{{ ch === ' ' ? '\u00A0' : ch }}</span>
        </h1>
        <h2 class="name">{{ C.name }}</h2>
        <p class="typed">{{ typed }}<i class="caret"></i></p>
        <div class="scroll"><span></span></div>
      </section>

      <!-- MEMORIES -->
      <section class="sec">
        <h3 class="title" v-reveal>Our little universe</h3>
        <div class="polas">
          <figure v-for="(m, i) in memories" :key="i" v-reveal class="pola" :style="{ '--r': (i % 2 ? 3 : -3) + (i * 0.7) + 'deg', '--d': i * 0.12 + 's' }">
            <div class="ph">
              <img :src="m.img" :alt="m.title" @error="$event.target.style.display = 'none'" />
              <span class="ph-fallback">♥</span>
            </div>
            <figcaption>
              <b>{{ m.title }}</b>
              <small>{{ m.date }}</small>
              <em>{{ m.text }}</em>
            </figcaption>
          </figure>
        </div>
      </section>

      <!-- QUOTES -->
      <section class="sec quote-sec">
        <div class="moon" v-reveal></div>
        <transition name="q" mode="out-in">
          <blockquote :key="qi">“{{ quotes[qi] }}”</blockquote>
        </transition>
        <p class="q-sig">for you, {{ C.nick }}</p>
      </section>

      <!-- REASONS -->
      <section class="sec">
        <h3 class="title" v-reveal>Why I love you</h3>
        <p class="sub" v-reveal>(touch each card)</p>
        <div class="cards">
          <div v-for="(r, i) in reasons" :key="i" v-reveal class="flip" tabindex="0" :style="{ '--d': i * 0.08 + 's' }"
               @click="$event.currentTarget.classList.toggle('on')" @keyup.enter="$event.currentTarget.classList.toggle('on')">
            <div class="face front">{{ r.q }}</div>
            <div class="face back">{{ r.a }}</div>
          </div>
        </div>
      </section>

      <!-- LETTER -->
      <section class="sec">
        <article class="letter" v-reveal>
          <p v-for="(p, i) in C.letter" :key="i" :class="{ first: i === 0 }">{{ p }}</p>
          <p class="sign">Forever yours,<br /><span>{{ C.from }}</span></p>
        </article>
      </section>

      <!-- WISH -->
      <section class="sec wish">
        <h3 class="title" v-reveal>Make a wish, {{ C.nick }}</h3>
        <button class="cake" @click="blow" :class="{ blown }" aria-label="Blow the candles">
          <div class="candles">
            <i v-for="n in 3" :key="n"><b class="flame"></b></i>
          </div>
          <div class="tier t1"></div><div class="tier t2"></div><div class="plate"></div>
        </button>
        <p v-if="!blown" class="sub">tap the cake to blow the candles</p>
        <transition name="q">
          <div v-if="blown" class="final">
            <h2>Happy Birthday, {{ C.name }}</h2>
            <p>May every wish you made come true. I love you.</p>
          </div>
        </transition>
      </section>
    </main>

    <canvas ref="fx" class="cv fx"></canvas>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Italiana&family=Cormorant+Garamond:ital,wght@0,400;0,600;1,400&family=Pinyon+Script&display=swap');

:root {
  --ink: #12061c; --plum: #2a0f3d; --rose: #ff5c8a; --blush: #ffb3c9;
  --gold: #ffd9a0; --cream: #fff3d6; --lav: #b79cff;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
html { scroll-behavior: smooth; }
body { background: var(--ink); color: var(--cream); font-family: 'Cormorant Garamond', serif; font-size: 20px; overflow-x: hidden; }
</style>

<style scoped>
.app { min-height: 100vh; position: relative; background: radial-gradient(120% 80% at 50% 0%, #3a1450 0%, var(--ink) 60%); }
.cv { position: fixed; inset: 0; pointer-events: none; z-index: 0; }
.cv.fx { z-index: 60; }
main { position: relative; z-index: 1; }

/* aurora */
.aurora { position: fixed; inset: 0; z-index: 0; filter: blur(70px); opacity: .55; overflow: hidden; }
.aurora i { position: absolute; width: 50vmax; height: 50vmax; border-radius: 50%; animation: drift 18s ease-in-out infinite alternate; }
.aurora i:nth-child(1) { background: #ff5c8a55; top: -15%; left: -10%; }
.aurora i:nth-child(2) { background: #b79cff55; bottom: -20%; right: -10%; animation-delay: -6s; }
.aurora i:nth-child(3) { background: #ffd9a033; top: 35%; left: 35%; animation-delay: -12s; }
@keyframes drift { to { transform: translate(8vw, 6vh) scale(1.25); } }

/* gate */
.gate { position: fixed; inset: 0; z-index: 50; display: grid; place-content: center; justify-items: center; gap: 2rem; text-align: center; background: radial-gradient(circle at 50% 40%, #3a1450, var(--ink) 70%); }
.gate-hint { font-family: 'Pinyon Script', cursive; font-size: clamp(2rem, 7vw, 3.2rem); color: var(--gold); animation: float 4s ease-in-out infinite; }
.gate-tap { font-style: italic; opacity: .6; letter-spacing: .08em; animation: blink 2s infinite; }
.env { position: relative; width: min(300px, 70vw); aspect-ratio: 3/2; border: 0; cursor: pointer; border-radius: 8px;
  background: linear-gradient(160deg, #ffd9a0, #f5b88a); box-shadow: 0 25px 60px #ff5c8a55, 0 0 0 1px #fff3d655 inset; animation: float 4s ease-in-out infinite .4s; transition: transform .5s; }
.env:hover { transform: scale(1.06) rotate(-1.5deg); }
.env-flap { position: absolute; inset: 0 0 auto 0; height: 62%; background: linear-gradient(#ffe6bd, #f2b07f); clip-path: polygon(0 0, 100% 0, 50% 100%); border-radius: 8px 8px 0 0; }
.env-seal { position: absolute; left: 50%; top: 52%; translate: -50% -50%; width: 62px; height: 62px; border-radius: 50%; display: grid; place-items: center;
  background: radial-gradient(circle at 35% 30%, #ff8fb0, #c2184f); color: #fff; font-size: 30px; box-shadow: 0 6px 18px #0006; animation: beat 1.3s infinite; }
@keyframes beat { 0%, 100% { scale: 1; } 15% { scale: 1.18; } 30% { scale: 1; } 45% { scale: 1.12; } }
@keyframes float { 50% { transform: translateY(-12px); } }
@keyframes blink { 50% { opacity: .15; } }
.gate-leave-active { transition: opacity 1.2s, transform 1.2s; }
.gate-leave-to { opacity: 0; transform: scale(1.4); }

/* music */
.music { position: fixed; top: 18px; right: 18px; z-index: 70; width: 46px; height: 46px; border-radius: 50%; border: 1px solid #fff3d655; background: #12061c99; backdrop-filter: blur(8px); display: flex; gap: 3px; align-items: flex-end; justify-content: center; padding: 14px 0; cursor: pointer; }
.music span { width: 3px; height: 6px; background: var(--gold); border-radius: 2px; }
.music.on span { animation: eq .8s ease-in-out infinite alternate; }
.music.on span:nth-child(2) { animation-delay: -.3s; } .music.on span:nth-child(3) { animation-delay: -.6s; } .music.on span:nth-child(4) { animation-delay: -.15s; }
@keyframes eq { to { height: 18px; } }

/* hero */
.hero { min-height: 100svh; display: grid; place-content: center; justify-items: center; text-align: center; padding: 2rem 1rem; }
.hb { font-family: 'Italiana', serif; font-weight: 400; font-size: clamp(2.6rem, 11vw, 7.5rem); letter-spacing: .02em; line-height: 1; }
.hb span { display: inline-block; opacity: 0; animation: drop 1s cubic-bezier(.2, 1.4, .4, 1) calc(var(--i) * .07s + .2s) forwards;
  background: linear-gradient(180deg, #fff3d6, #ffd9a0 60%, #ff9ec0); -webkit-background-clip: text; background-clip: text; color: transparent; }
@keyframes drop { from { opacity: 0; transform: translateY(-80px) rotate(-14deg) scale(.6); filter: blur(8px); } to { opacity: 1; transform: none; filter: none; } }
.name { font-family: 'Pinyon Script', cursive; font-weight: 400; font-size: clamp(4.5rem, 18vw, 12rem); line-height: 1.1; margin-top: -.05em; color: var(--rose);
  text-shadow: 0 0 30px #ff5c8a99, 0 0 80px #ff5c8a55; opacity: 0; animation: nameIn 1.8s ease 1.4s forwards; }
@keyframes nameIn { from { opacity: 0; letter-spacing: .3em; filter: blur(14px); } to { opacity: 1; letter-spacing: 0; filter: none; } }
.typed { margin-top: 1.2rem; font-style: italic; font-size: clamp(1.1rem, 3.6vw, 1.6rem); min-height: 2.2em; max-width: 30ch; color: var(--blush); }
.caret { display: inline-block; width: 2px; height: 1em; background: var(--gold); margin-left: 3px; vertical-align: -2px; animation: blink 1s steps(1) infinite; }
.scroll { margin-top: 3rem; width: 26px; height: 44px; border: 1.5px solid #fff3d688; border-radius: 14px; position: relative; opacity: 0; animation: nameIn 1s 4s forwards; }
.scroll span { position: absolute; left: 50%; top: 8px; width: 4px; height: 8px; margin-left: -2px; border-radius: 2px; background: var(--gold); animation: wheel 1.6s infinite; }
@keyframes wheel { to { transform: translateY(16px); opacity: 0; } }

/* sections */
.sec { padding: 6rem 1.2rem; max-width: 1100px; margin: 0 auto; text-align: center; }
.title { font-family: 'Pinyon Script', cursive; font-weight: 400; font-size: clamp(2.8rem, 9vw, 5rem); color: var(--gold); margin-bottom: 2.5rem; }
.sub { opacity: .65; font-style: italic; margin: -1.6rem 0 2rem; }
.rv { opacity: 0; transform: translateY(40px) scale(.96); transition: opacity 1s var(--d, 0s), transform 1s var(--d, 0s) cubic-bezier(.2, .8, .2, 1); }
.rv.in { opacity: 1; transform: none; }

/* polaroids */
.polas { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 2.4rem 2rem; padding: 0 .5rem; }
.pola { background: #fff3e4; color: #3b1b2c; padding: 12px 12px 18px; border-radius: 4px; box-shadow: 0 20px 50px #0008; rotate: var(--r); text-align: left; transition: rotate .4s, scale .4s; }
.pola:hover { rotate: 0deg; scale: 1.05; z-index: 2; }
.ph { position: relative; aspect-ratio: 1; background: linear-gradient(135deg, #ffb3c9, #b79cff); overflow: hidden; display: grid; place-items: center; }
.ph img { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; z-index: 1; }
.ph-fallback { font-size: 4rem; color: #fff9; }
figcaption { padding-top: .9rem; display: grid; gap: .15rem; }
figcaption b { font-family: 'Pinyon Script', cursive; font-size: 1.9rem; font-weight: 400; color: #b0174d; }
figcaption small { opacity: .55; font-size: .85rem; }
figcaption em { font-size: 1rem; line-height: 1.35; }

/* quotes */
.quote-sec { min-height: 80vh; display: grid; place-content: center; position: relative; }
.moon { position: absolute; left: 50%; top: 50%; translate: -50% -50%; width: min(70vw, 420px); aspect-ratio: 1; border-radius: 50%;
  background: radial-gradient(circle at 35% 35%, #fff3d633, #ffb3c911 60%, transparent 70%); box-shadow: 0 0 120px #ffb3c922; animation: pulse 6s ease-in-out infinite; }
@keyframes pulse { 50% { scale: 1.1; } }
blockquote { position: relative; font-family: 'Italiana', serif; font-size: clamp(1.7rem, 6vw, 3.2rem); line-height: 1.25; max-width: 22ch; margin: 0 auto; }
.q-sig { position: relative; margin-top: 1.5rem; font-family: 'Pinyon Script', cursive; font-size: 1.8rem; color: var(--rose); }
.q-enter-active, .q-leave-active { transition: opacity .9s, transform .9s, filter .9s; }
.q-enter-from { opacity: 0; transform: translateY(24px); filter: blur(8px); }
.q-leave-to { opacity: 0; transform: translateY(-24px); filter: blur(8px); }

/* flip cards */
.cards { display: grid; grid-template-columns: repeat(auto-fit, minmax(210px, 1fr)); gap: 1.4rem; perspective: 1000px; }
.flip { position: relative; height: 170px; cursor: pointer; transform-style: preserve-3d; transition: transform .8s cubic-bezier(.3, 1.3, .5, 1), opacity 1s var(--d), translate 1s var(--d); outline: none; }
.flip.rv.in.on, .flip.on { transform: rotateY(180deg); }
.face { position: absolute; inset: 0; display: grid; place-items: center; padding: 1.2rem; border-radius: 18px; backface-visibility: hidden; border: 1px solid #fff3d633; }
.front { font-family: 'Italiana', serif; font-size: 1.7rem; background: linear-gradient(145deg, #ffffff14, #ff5c8a22); backdrop-filter: blur(6px); }
.back { transform: rotateY(180deg); background: linear-gradient(145deg, #ff5c8a, #8e2d9c); font-style: italic; font-size: 1.2rem; line-height: 1.35; }

/* letter */
.letter { max-width: 640px; margin: 0 auto; text-align: left; padding: 3rem 2.2rem; border-radius: 6px; color: #3b1b2c; line-height: 1.6;
  background: repeating-linear-gradient(#fff3e4 0 31px, #f3d9cc 32px); box-shadow: 0 30px 80px #000a; rotate: -1deg; }
.letter p { margin-bottom: 1rem; font-size: 1.15rem; }
.letter .first { font-family: 'Pinyon Script', cursive; font-size: 2.4rem; color: #b0174d; }
.sign { text-align: right; }
.sign span { font-family: 'Pinyon Script', cursive; font-size: 2.2rem; color: #b0174d; }

/* cake */
.wish { padding-bottom: 8rem; }
.cake { position: relative; background: none; border: 0; cursor: pointer; width: 230px; height: 230px; margin: 1rem auto 2rem; display: block; }
.plate { position: absolute; bottom: 0; left: -10%; width: 120%; height: 14px; border-radius: 50%; background: #fff3d6; box-shadow: 0 10px 30px #0008; }
.tier { position: absolute; left: 50%; translate: -50% 0; border-radius: 12px 12px 6px 6px; }
.t1 { bottom: 12px; width: 230px; height: 70px; background: linear-gradient(#ff8fb0 0 22%, #e0457a 22%); }
.t2 { bottom: 82px; width: 160px; height: 62px; background: linear-gradient(#ffe6bd 0 24%, #f2a65e 24%); }
.candles { position: absolute; bottom: 144px; left: 50%; translate: -50% 0; display: flex; gap: 22px; }
.candles i { position: relative; width: 9px; height: 34px; background: repeating-linear-gradient(60deg, #fff 0 6px, #ff9ec0 6px 12px); border-radius: 2px; }
.flame { position: absolute; left: 50%; top: -26px; width: 14px; height: 24px; margin-left: -7px; border-radius: 50% 50% 50% 50% / 62% 62% 38% 38%;
  background: radial-gradient(#fff 15%, #ffd9a0 45%, #ff8a3d); box-shadow: 0 0 22px 8px #ffb86a88; transform-origin: bottom; animation: flick .18s infinite alternate; transition: opacity .4s, transform .4s; }
@keyframes flick { to { transform: scale(.92, 1.08) rotate(3deg); } }
.blown .flame { opacity: 0; transform: translateY(-30px) scale(.2); animation: none; }
.final { margin-top: 1rem; }
.final h2 { font-family: 'Pinyon Script', cursive; font-weight: 400; font-size: clamp(2.8rem, 10vw, 5.5rem); color: var(--rose); text-shadow: 0 0 40px #ff5c8a88; }
.final p { font-style: italic; font-size: 1.4rem; color: var(--blush); }

@media (prefers-reduced-motion: reduce) {
  * { animation-duration: .01ms !important; transition-duration: .01ms !important; }
}
</style>
