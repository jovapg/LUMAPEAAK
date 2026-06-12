<script setup>
import { ref, computed } from 'vue'
import content from '../data/content.json'

const { videos } = content
const items = videos.items
const n = items.length
const active = ref(0)
const current = computed(() => items[active.value])

function go(i) { active.value = (i + n) % n }
function next() { go(active.value + 1) }
function prev() { go(active.value - 1) }

// shortest signed distance from active (wraps around)
function offsetFor(i) {
  let o = i - active.value
  if (o > n / 2) o -= n
  if (o < -n / 2) o += n
  return o
}

function cardStyle(i) {
  const o = offsetFor(i)
  const abs = Math.abs(o)
  if (abs > 2) {
    return { opacity: 0, pointerEvents: 'none', transform: `translateX(calc(-50% + ${o * 60}px)) scale(0.5)` }
  }
  const spacing = 200
  const scale = o === 0 ? 1 : 0.82 - (abs - 1) * 0.12
  const rot = o === 0 ? 0 : (o > 0 ? -22 : 22)
  return {
    transform: `translateX(calc(-50% + ${o * spacing}px)) scale(${scale}) rotateY(${rot}deg)`,
    zIndex: 20 - abs,
    opacity: o === 0 ? 1 : abs === 1 ? 0.9 : 0.55,
    filter: o === 0 ? 'none' : `brightness(${abs === 1 ? 0.6 : 0.4})`,
  }
}
</script>

<template>
  <section class="seccion seccion--dark alt-shade vshow">
    <div class="contenedor">
      <div class="center-head">
        <span class="eyebrow">{{ videos.eyebrow }}</span>
        <h2 class="titulo-seccion">{{ videos.title }}</h2>
        <p class="subtitulo-seccion">{{ videos.subtitle }}</p>
      </div>

      <div class="vshow__stage">
        <button class="vshow__arrow vshow__arrow--prev" type="button" aria-label="Previous video" @click="prev">‹</button>

        <div class="vshow__track">
          <div
            v-for="(v, i) in items"
            :key="v.src"
            class="vshow__card"
            :class="{ 'is-active': i === active }"
            :style="cardStyle(i)"
            @click="i === active ? null : go(i)"
          >
            <!-- only the active card streams video; the rest show their poster -->
            <video
              v-if="i === active"
              :key="v.src"
              class="vshow__media"
              :src="v.src"
              :poster="v.poster"
              autoplay
              muted
              playsinline
              preload="metadata"
              @ended="next"
            ></video>
            <img v-else class="vshow__media" :src="v.poster" :alt="v.title" loading="lazy" />

            <span v-if="i === active" class="vshow__tag">{{ v.tag }}</span>
            <div v-if="i === active" class="vshow__caption">{{ v.title }}</div>
          </div>
        </div>

        <button class="vshow__arrow vshow__arrow--next" type="button" aria-label="Next video" @click="next">›</button>
      </div>

      <div class="vshow__dots">
        <button
          v-for="(v, i) in items"
          :key="v.src"
          type="button"
          class="vshow__dot"
          :class="{ 'is-active': i === active }"
          :aria-label="v.title"
          @click="go(i)"
        ></button>
      </div>
    </div>
  </section>
</template>

<style scoped>
.vshow__stage {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
}
.vshow__track {
  position: relative;
  flex: 1;
  max-width: 760px;
  height: 480px;
  perspective: 1500px;
  transform-style: preserve-3d;
}
.vshow__card {
  position: absolute;
  top: 0;
  left: 50%;
  width: 270px;
  height: 480px;
  border-radius: 18px;
  overflow: hidden;
  background: #06141c;
  box-shadow: 0 30px 70px rgba(0, 0, 0, 0.55);
  border: 1px solid var(--color-dark-borde);
  cursor: pointer;
  transition: transform 0.5s ease, opacity 0.5s ease, filter 0.5s ease;
  will-change: transform;
}
.vshow__card.is-active { cursor: default; border-color: rgba(89, 173, 71, 0.5); }
.vshow__media { width: 100%; height: 100%; object-fit: cover; display: block; }
.vshow__tag {
  position: absolute;
  top: 14px;
  left: 14px;
  font-family: var(--fuente-display);
  text-transform: uppercase;
  letter-spacing: 0.06em;
  font-size: 0.7rem;
  font-weight: 600;
  color: #fff;
  background: var(--color-acento);
  padding: 5px 12px;
  border-radius: 50px;
}
.vshow__caption {
  position: absolute;
  left: 0; right: 0; bottom: 0;
  padding: 36px 16px 16px;
  font-family: var(--fuente-display);
  text-transform: uppercase;
  letter-spacing: 0.03em;
  font-size: 0.98rem;
  color: #fff;
  background: linear-gradient(to top, rgba(6, 20, 28, 0.92), transparent);
}
.vshow__arrow {
  flex-shrink: 0;
  width: 52px;
  height: 52px;
  border-radius: 50%;
  border: 1px solid var(--color-dark-borde);
  background: rgba(255, 255, 255, 0.06);
  color: #fff;
  font-size: 1.8rem;
  line-height: 1;
  cursor: pointer;
  z-index: 30;
  transition: var(--transicion);
}
.vshow__arrow:hover { background: var(--color-acento); border-color: var(--color-acento); transform: scale(1.06); }

.vshow__dots {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 28px;
}
.vshow__dot {
  width: 9px;
  height: 9px;
  border-radius: 50%;
  border: none;
  padding: 0;
  cursor: pointer;
  background: rgba(255, 255, 255, 0.3);
  transition: var(--transicion);
}
.vshow__dot.is-active { background: var(--color-acento); width: 26px; border-radius: 5px; }

@media (max-width: 700px) {
  .vshow__track { height: 420px; }
  .vshow__card { width: 220px; height: 390px; }
  .vshow__arrow { width: 42px; height: 42px; font-size: 1.4rem; }
}
@media (max-width: 480px) {
  .vshow__track { height: 380px; }
  .vshow__card { width: 190px; height: 338px; }
}
</style>
