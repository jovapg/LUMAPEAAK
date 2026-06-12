<script setup>
import { ref, computed, watch, onMounted, onBeforeUnmount, nextTick } from 'vue'
import content from '../data/content.json'

const { reviews } = content
const items = ref(reviews.items) // sample data; replaced by live Google reviews if configured
const liveRating = ref(reviews.rating)
const liveCount = ref(reviews.count)

const perView = ref(8)
const rowEl = ref(null)
const slots = ref([]) // item indices currently on screen (one per visible card)
let bringPtr = 0 // next review index to bring in
let slotPtr = 0 // which slot gets replaced next
let timer

const reviewAt = (i) => items.value[i] || {}

// how many cards are visible. When everything fits we keep one review in
// reserve so there's always a fresh one to rotate in (one at a time).
const count = computed(() => {
  const len = items.value.length
  if (!len) return 0
  return len <= perView.value ? Math.max(1, len - 1) : perView.value
})

function initSlots() {
  const len = items.value.length
  const c = count.value
  slots.value = Array.from({ length: c }, (_, i) => i % len)
  bringPtr = c % len
  slotPtr = 0
  nextTick(layoutMasonry)
}

// replace exactly ONE card with the next (unseen) review
function rotateForward() {
  const len = items.value.length
  if (len <= 1 || !count.value) return
  slots.value[slotPtr] = bringPtr
  bringPtr = (bringPtr + 1) % len
  slotPtr = (slotPtr + 1) % count.value
}
function rotateBackward() {
  const len = items.value.length
  if (len <= 1 || !count.value) return
  slotPtr = (slotPtr - 1 + count.value) % count.value
  slots.value[slotPtr] = bringPtr
  bringPtr = (bringPtr - 1 + len) % len
}
const next = () => { rotateForward(); start() }
const prev = () => { rotateBackward(); start() }

function start() {
  stop()
  timer = setInterval(rotateForward, 3000)
}
function stop() {
  if (timer) clearInterval(timer)
}

/* JS masonry: measure each card and set its grid row-span so cards pack
   tightly with no gaps. Some cards are wider (col-span 2). */
function layoutMasonry(el) {
  const row = el instanceof Element ? el : rowEl.value
  if (!row) return
  const rowUnit = parseFloat(getComputedStyle(row).gridAutoRows) || 4
  row.querySelectorAll('.rev-card').forEach((card) => {
    const inner = card.querySelector('.rev-card__inner:not(.card-fade-leave-active)')
      || card.querySelector('.rev-card__inner')
    if (!inner) return
    const mb = parseFloat(getComputedStyle(inner).marginBottom) || 0
    const h = inner.offsetHeight + mb
    card.style.gridRowEnd = `span ${Math.max(1, Math.ceil(h / rowUnit))}`
  })
}

function computePerView() {
  const w = window.innerWidth
  perView.value = w <= 620 ? 4 : w <= 980 ? 6 : 8
  initSlots()
}

/* ---- live Google reviews via Featurable (free, no API key, updates daily) ---- */
function relativeDate(iso) {
  if (!iso) return ''
  const then = new Date(iso).getTime()
  const days = Math.round((Date.now() - then) / 86400000)
  if (days <= 1) return 'a day ago'
  if (days < 30) return `${days} days ago`
  const months = Math.round(days / 30)
  if (months < 12) return `${months} month${months > 1 ? 's' : ''} ago`
  const years = Math.round(months / 12)
  return `${years} year${years > 1 ? 's' : ''} ago`
}

async function loadLiveReviews() {
  const id = reviews.featurableWidgetId
  if (!id) return // not configured yet -> keep sample reviews
  try {
    const res = await fetch(`https://api.featurable.com/v1/widgets/${id}`)
    if (!res.ok) return
    const data = await res.json()
    const list = data.reviews || data.data?.reviews || []
    const mapped = list
      .filter((r) => r.comment && r.comment.trim())
      .map((r) => ({
        name: r.reviewer?.displayName || 'Google user',
        rating: r.starRating || 5,
        date: relativeDate(r.createTime),
        text: r.comment,
        avatar: r.reviewer?.profilePhotoUrl || '',
      }))
    if (typeof data.averageRating === 'number') liveRating.value = data.averageRating
    if (typeof data.totalReviewCount === 'number') liveCount.value = data.totalReviewCount
    if (mapped.length) {
      items.value = mapped
      initSlots()
    }
  } catch (e) {
    /* network/CORS issue -> silently keep sample reviews */
  }
}

onMounted(async () => {
  await nextTick()
  computePerView()
  layoutMasonry()
  if (document.fonts && document.fonts.ready) document.fonts.ready.then(layoutMasonry)
  start()
  window.addEventListener('resize', computePerView)
  loadLiveReviews()
})
onBeforeUnmount(() => {
  stop()
  window.removeEventListener('resize', computePerView)
})

/* ---- avatar helpers ---- */
const palette = ['#0076b2', '#59ad47', '#e08a1e', '#7a5cc7', '#d4604f', '#2a9d8f']
function initials(name) {
  return name.split(' ').map((w) => w[0]).join('').slice(0, 2).toUpperCase()
}
function avatarColor(name) {
  let h = 0
  for (const ch of name) h = (h + ch.charCodeAt(0)) % palette.length
  return palette[h]
}
</script>

<template>
  <section id="reviews" class="reviews">
    <!-- brand-colored decorative shapes -->
    <span class="deco deco--ring" aria-hidden="true"></span>
    <span class="deco deco--halfL" aria-hidden="true"></span>
    <span class="deco deco--halfR" aria-hidden="true"></span>
    <span class="deco deco--dots" aria-hidden="true"></span>
    <span class="deco deco--dot1" aria-hidden="true"></span>
    <span class="deco deco--dot2" aria-hidden="true"></span>

    <div class="contenedor">
      <div class="reviews__head">
        <span class="eyebrow reviews__eyebrow">{{ reviews.eyebrow }}</span>
        <h2 class="reviews__title">{{ reviews.title }}</h2>
        <p class="reviews__sub">{{ reviews.subtitle }}</p>

        <a :href="reviews.googleUrl" target="_blank" rel="noopener" class="reviews__badge">
          <svg class="reviews__g" viewBox="0 0 48 48" aria-hidden="true">
            <path fill="#4285F4" d="M45.12 24.5c0-1.56-.14-3.06-.4-4.5H24v8.51h11.84c-.51 2.75-2.06 5.08-4.39 6.64v5.52h7.11c4.16-3.83 6.56-9.47 6.56-16.17z"/>
            <path fill="#34A853" d="M24 46c5.94 0 10.92-1.97 14.56-5.33l-7.11-5.52c-1.97 1.32-4.49 2.1-7.45 2.1-5.73 0-10.58-3.87-12.31-9.07H4.34v5.7C7.96 41.07 15.4 46 24 46z"/>
            <path fill="#FBBC05" d="M11.69 28.18C11.25 26.86 11 25.45 11 24s.25-2.86.69-4.18v-5.7H4.34A21.98 21.98 0 0 0 2 24c0 3.55.85 6.91 2.34 9.88l7.35-5.7z"/>
            <path fill="#EA4335" d="M24 10.75c3.23 0 6.13 1.11 8.41 3.29l6.31-6.31C34.91 4.18 29.93 2 24 2 15.4 2 7.96 6.93 4.34 14.12l7.35 5.7c1.73-5.2 6.58-9.07 12.31-9.07z"/>
          </svg>
          <span class="reviews__badge-text">
            <strong>{{ liveRating.toFixed(1) }}</strong>
            <span class="reviews__badge-stars">★★★★★</span>
            <small>{{ liveCount }} Google reviews</small>
          </span>
        </a>
      </div>

      <!-- living mosaic: one card swaps at a time, each with its own fade -->
      <div class="reviews__stage" @mouseenter="stop" @mouseleave="start">
        <div class="reviews__row" ref="rowEl">
          <article v-for="(itemIdx, pos) in slots" :key="pos" class="rev-card">
            <Transition name="card-fade" @after-enter="layoutMasonry">
              <div class="rev-card__inner" :key="itemIdx">
                <span class="rev-card__quote" aria-hidden="true">&rdquo;</span>
                <div class="rev-card__stars" :aria-label="`${reviewAt(itemIdx).rating} out of 5 stars`">
                  <span v-for="n in 5" :key="n" :class="{ on: n <= reviewAt(itemIdx).rating }">★</span>
                </div>
                <p class="rev-card__text">{{ reviewAt(itemIdx).text }}</p>
                <div class="rev-card__person">
                  <img v-if="reviewAt(itemIdx).avatar" :src="reviewAt(itemIdx).avatar" :alt="reviewAt(itemIdx).name" class="rev-card__avatar" referrerpolicy="no-referrer" />
                  <span
                    v-else
                    class="rev-card__avatar rev-card__avatar--initials"
                    :style="{ background: avatarColor(reviewAt(itemIdx).name) }"
                  >{{ initials(reviewAt(itemIdx).name) }}</span>
                  <span class="rev-card__meta">
                    <strong>{{ reviewAt(itemIdx).name }}</strong>
                    <small>{{ reviewAt(itemIdx).date }}</small>
                  </span>
                </div>
              </div>
            </Transition>
          </article>
        </div>
      </div>

      <div class="reviews__controls">
        <button class="reviews__arrow" @click="prev" aria-label="Previous review">‹</button>
        <button class="reviews__arrow" @click="next" aria-label="Next review">›</button>
      </div>
    </div>
  </section>
</template>

<style scoped>
.reviews {
  position: relative;
  overflow: hidden;
  background: linear-gradient(165deg, var(--color-dark-2) 0%, var(--color-dark) 100%);
  color: #fff;
  padding: 60px 0;
}
.reviews .contenedor { position: relative; z-index: 2; }

/* ---------- brand-colored decorative shapes ---------- */
.deco { position: absolute; z-index: 1; pointer-events: none; }
.deco--ring {
  width: 260px; height: 260px; border-radius: 50%;
  border: 16px solid var(--color-acento);
  opacity: 0.45;
  top: -90px; right: -70px;
}
.deco--halfL {
  width: 140px; height: 280px;
  background: var(--color-primario);
  border-radius: 0 280px 280px 0;
  left: 0; top: 56%; transform: translateY(-50%);
  opacity: 0.4;
}
.deco--halfR {
  width: 110px; height: 220px;
  background: var(--color-acento);
  border-radius: 220px 0 0 220px;
  right: 0; bottom: 8%;
  opacity: 0.4;
}
.deco--dots {
  width: 130px; height: 130px;
  left: 7%; bottom: 38px;
  background-image: radial-gradient(var(--color-acento) 2px, transparent 2.5px);
  background-size: 18px 18px;
  opacity: 0.4;
}
.deco--dot1 { width: 18px; height: 18px; border-radius: 50%; background: var(--color-acento); top: 70px; left: 16%; opacity: 0.7; }
.deco--dot2 { width: 12px; height: 12px; border-radius: 50%; background: var(--color-primario); top: 130px; right: 22%; opacity: 0.7; }

/* ---------- header ---------- */
.reviews__head { text-align: center; max-width: 640px; margin: 0 auto 30px; }
.reviews__eyebrow { color: var(--color-acento); margin-bottom: 10px; }
.reviews__title {
  font-family: var(--fuente-display);
  text-transform: uppercase;
  font-size: clamp(1.9rem, 4vw, 3rem);
  font-weight: 700;
  line-height: 1.05;
  margin-bottom: 14px;
}
.reviews__sub { color: rgba(255, 255, 255, 0.78); font-size: 1.05rem; margin-bottom: 22px; }
.reviews__badge {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.18);
  border-radius: 999px;
  padding: 9px 18px 9px 14px;
  transition: var(--transicion);
}
.reviews__badge:hover { background: rgba(255, 255, 255, 0.14); }
.reviews__g { width: 26px; height: 26px; }
.reviews__badge-text { display: flex; align-items: center; gap: 8px; }
.reviews__badge-text strong { font-size: 1.15rem; color: #fff; }
.reviews__badge-stars { color: #f0b53f; letter-spacing: 1px; }
.reviews__badge-text small { color: rgba(255, 255, 255, 0.7); font-size: 0.82rem; }

/* ---------- cards: real masonry mosaic (JS sets row spans) ---------- */
.reviews__stage { position: relative; }
.reviews__row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-auto-rows: 4px;        /* fine unit; JS spans rows per card height */
  grid-auto-flow: row dense;  /* backfill holes -> tight cluster */
  column-gap: 16px;
  row-gap: 0;
  max-width: 1000px;
  margin: 0 auto;
}
/* some cards are wider -> varied tile sizes like a photo collage */
.rev-card:nth-child(7n + 1) { grid-column: span 2; }
.rev-card:nth-child(7n + 5) { grid-column: span 2; }

.rev-card { position: relative; min-width: 0; }
.rev-card__inner {
  position: relative;
  margin-bottom: 14px;
  background: #fff;
  color: var(--color-texto);
  border-radius: 14px;
  padding: 18px 18px 16px;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.3);
  overflow: hidden;
  transition: transform 0.22s ease, box-shadow 0.22s ease;
}
/* grow on hover */
.rev-card:hover { z-index: 5; }
.rev-card__inner:hover {
  transform: scale(1.06);
  box-shadow: 0 20px 44px rgba(0, 0, 0, 0.42);
}

/* per-card cross-fade: leaving content is taken out of flow so the card
   doesn't collapse; only the one changing card animates */
.card-fade-enter-active,
.card-fade-leave-active { transition: opacity 0.55s ease; }
.card-fade-enter-from,
.card-fade-leave-to { opacity: 0; }
.card-fade-leave-active { position: absolute; top: 0; left: 0; width: 100%; }

.rev-card__quote {
  position: absolute;
  top: 0; right: 12px;
  font-family: var(--fuente-display);
  font-size: 3.2rem;
  line-height: 1;
  color: rgba(89, 173, 71, 0.22);
}
.rev-card__stars { color: #f0b53f; font-size: 0.86rem; letter-spacing: 1.5px; margin-bottom: 9px; }
.rev-card__stars span { color: #e3e3e3; }
.rev-card__stars span.on { color: #f0b53f; }
.rev-card__text {
  color: var(--color-texto-suave);
  font-size: 0.85rem;
  line-height: 1.5;
  margin-bottom: 16px;
}
.rev-card__person { display: flex; align-items: center; gap: 10px; }
.rev-card__avatar {
  width: 36px; height: 36px;
  border-radius: 50%;
  object-fit: cover;
  flex-shrink: 0;
}
.rev-card__avatar--initials {
  display: flex; align-items: center; justify-content: center;
  color: #fff; font-weight: 700; font-size: 0.85rem;
}
.rev-card__meta { display: flex; flex-direction: column; line-height: 1.3; }
.rev-card__meta strong { color: var(--color-texto); font-size: 0.88rem; }
.rev-card__meta small { color: var(--color-texto-suave); font-size: 0.76rem; }

/* ---------- controls ---------- */
.reviews__controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 14px;
  margin-top: 22px;
}
.reviews__arrow {
  width: 40px; height: 40px;
  border-radius: 50%;
  border: 1px solid rgba(255, 255, 255, 0.25);
  background: rgba(255, 255, 255, 0.08);
  color: #fff;
  font-size: 1.4rem;
  line-height: 1;
  cursor: pointer;
  transition: var(--transicion);
}
.reviews__arrow:hover { background: var(--color-acento); border-color: var(--color-acento); color: #fff; }

@media (max-width: 980px) {
  .reviews__row { grid-template-columns: repeat(3, 1fr); max-width: 720px; }
  .deco--ring, .deco--dots, .deco--dot1, .deco--dot2 { display: none; }
}
@media (max-width: 620px) {
  .reviews { padding: 48px 0; }
  .reviews__row { grid-template-columns: repeat(2, 1fr); column-gap: 12px; }
  .rev-card__inner { margin-bottom: 12px; }
  .deco--halfL, .deco--halfR { display: none; }
}
</style>
