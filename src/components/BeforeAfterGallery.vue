<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import content from '../data/content.json'

const { beforeAfter } = content
const items = beforeAfter.items
const n = items.length

const track = ref(null)
const activeDot = ref(0)

let pos = 0        // current continuous position
let target = 0     // where we're easing toward
let hovered = -1
let paused = false
let raf = 0
const AUTO = 0.0045 // slow continuous drift (cards per frame)
const reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches

function shortestDelta(toIndex) {
  let d = ((toIndex - pos) % n + n) % n
  if (d > n / 2) d -= n
  return d
}
function go(i) { target = pos + shortestDelta(i) }
function next() { target += 1 }
function prev() { target -= 1 }
function setHover(i) { hovered = i }
function clearHover() { hovered = -1 }

function layout() {
  const cards = track.value && track.value.children
  if (!cards) return
  for (let i = 0; i < cards.length; i++) {
    let o = ((i - pos) % n + n) % n
    if (o > n / 2) o -= n
    const card = cards[i]
    const abs = Math.abs(o)
    if (abs > 3.2) {
      card.style.opacity = '0'
      card.style.pointerEvents = 'none'
      card.style.transform = 'translateX(-50%) scale(0.4)'
      continue
    }
    const isHover = i === hovered
    const spacing = 200
    let scale = 1 - abs * 0.04
    let extraZ = 0
    if (isHover) { scale += 0.16; extraZ = 110 }
    const rotY = isHover ? 0 : -o * 20
    const tz = -abs * 110 + extraZ
    const ty = isHover ? 0 : abs * 26
    card.style.opacity = abs > 2.6 ? String(1 - (abs - 2.6) / 0.6) : '1'
    card.style.pointerEvents = 'auto'
    card.style.zIndex = isHover ? '60' : String(30 - Math.round(abs))
    card.style.transform =
      `translateX(calc(-50% + ${o * spacing}px)) translateY(${ty}px) translateZ(${tz}px) rotateY(${rotY}deg) scale(${scale})`
  }
}

function frame() {
  if (!paused && !reduce && hovered === -1) target += AUTO
  pos += (target - pos) * 0.09
  if (pos > n) { pos -= n; target -= n }
  if (pos < -n) { pos += n; target += n }
  const a = ((Math.round(pos) % n) + n) % n
  if (a !== activeDot.value) activeDot.value = a
  layout()
  raf = requestAnimationFrame(frame)
}

onMounted(() => { layout(); raf = requestAnimationFrame(frame) })
onBeforeUnmount(() => cancelAnimationFrame(raf))
</script>

<template>
  <section id="before-after" class="ba seccion seccion--dark">
    <div class="contenedor">
      <div class="center-head">
        <span class="eyebrow">{{ beforeAfter.eyebrow }}</span>
        <h2 class="titulo-seccion">{{ beforeAfter.title }}</h2>
        <p class="subtitulo-seccion">{{ beforeAfter.subtitle }}</p>
      </div>
    </div>

    <div class="ba__stage" @mouseenter="paused = true" @mouseleave="paused = false">
      <button class="ba__arrow ba__arrow--prev" type="button" aria-label="Previous photo" @click="prev">‹</button>

      <div class="ba__track" ref="track">
        <div
          v-for="(src, i) in items"
          :key="src"
          class="ba__card"
          @mouseenter="setHover(i)"
          @mouseleave="clearHover"
          @click="go(i)"
        >
          <img class="ba__img" :src="src" :alt="`LumaPeak before and after — job ${i + 1}`" loading="lazy" decoding="async" />
          <span class="ba__badge">Before &amp; After</span>
        </div>
      </div>

      <button class="ba__arrow ba__arrow--next" type="button" aria-label="Next photo" @click="next">›</button>
    </div>

    <div class="contenedor">
      <div class="ba__dots">
        <button
          v-for="(src, i) in items"
          :key="src"
          type="button"
          class="ba__dot"
          :class="{ 'is-active': i === activeDot }"
          :aria-label="`Go to photo ${i + 1}`"
          @click="go(i)"
        ></button>
      </div>
    </div>
  </section>
</template>

<style scoped>
.ba { overflow: hidden; padding: 40px 0 36px; }
.ba .subtitulo-seccion { margin-bottom: 14px; }
.ba__stage {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 100%;
}
.ba__track {
  position: relative;
  flex: 1;
  max-width: 1240px;
  height: 410px;
  perspective: 1300px;
  transform-style: preserve-3d;
}
.ba__card {
  position: absolute;
  top: 34px;
  left: 50%;
  width: 380px;
  height: 340px;
  border-radius: 18px;
  overflow: hidden;
  background: #06141c;
  box-shadow: 0 24px 60px rgba(0, 0, 0, 0.5);
  border: 1px solid var(--color-dark-borde);
  cursor: pointer;
  transition: box-shadow 0.3s ease, border-color 0.3s ease;
  will-change: transform;
}
.ba__card:hover { box-shadow: 0 34px 80px rgba(0, 0, 0, 0.62); border-color: rgba(89, 173, 71, 0.6); }
.ba__img { width: 100%; height: 100%; object-fit: cover; display: block; }
.ba__badge {
  position: absolute;
  top: 14px;
  left: 50%;
  transform: translateX(-50%);
  font-family: var(--fuente-display);
  text-transform: uppercase;
  letter-spacing: 0.08em;
  font-size: 0.72rem;
  font-weight: 600;
  color: #fff;
  background: rgba(89, 173, 71, 0.92);
  padding: 6px 16px;
  border-radius: 50px;
  box-shadow: 0 4px 14px rgba(0, 0, 0, 0.4);
  white-space: nowrap;
}
.ba__arrow {
  position: absolute;
  z-index: 40;
  flex-shrink: 0;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  border: 1px solid var(--color-dark-borde);
  background: rgba(10, 24, 32, 0.6);
  backdrop-filter: blur(6px);
  color: #fff;
  font-size: 1.7rem;
  line-height: 1;
  cursor: pointer;
  transition: var(--transicion);
}
.ba__arrow--prev { left: 18px; }
.ba__arrow--next { right: 18px; }
.ba__arrow:hover { background: var(--color-acento); border-color: var(--color-acento); transform: scale(1.08); }

.ba__dots { display: flex; justify-content: center; gap: 10px; margin-top: 14px; }
.ba__dot {
  width: 9px; height: 9px;
  border-radius: 50%;
  border: none;
  padding: 0;
  cursor: pointer;
  background: rgba(255, 255, 255, 0.3);
  transition: var(--transicion);
}
.ba__dot.is-active { background: var(--color-acento); width: 26px; border-radius: 5px; }

@media (max-width: 760px) {
  .ba__track { height: 360px; }
  .ba__card { width: 280px; height: 250px; top: 40px; }
  .ba__arrow { width: 42px; height: 42px; font-size: 1.4rem; }
}
@media (max-width: 480px) {
  .ba__track { height: 320px; }
  .ba__card { width: 230px; height: 210px; }
}
</style>
