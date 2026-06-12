<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import content from '../data/content.json'

const { company } = content
const open = ref(false)
const activeId = ref('top')
const links = [
  { name: 'Home', to: '#top' },
  { name: 'Services', to: '#services' },
  { name: 'Service Areas', to: '#service-areas' },
  { name: 'About', to: '#about' },
]

let observer
onMounted(() => {
  // scroll-spy: highlight the link of the section currently in view
  const els = links
    .map((l) => document.getElementById(l.to.slice(1)))
    .filter(Boolean)
  observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((e) => {
        if (e.isIntersecting) activeId.value = e.target.id
      })
    },
    { rootMargin: '-45% 0px -50% 0px', threshold: 0 }
  )
  els.forEach((el) => observer.observe(el))
})
onBeforeUnmount(() => {
  if (observer) observer.disconnect()
})
</script>

<template>
  <!-- slim utility bar — scrolls away naturally (not sticky) -->
  <div class="topbar">
    <div class="contenedor topbar__inner">
      <span class="topbar__tag">★ Licensed &amp; Insured · Serving the Greater Seattle Area</span>
      <div class="topbar__right">
        <a :href="`mailto:${company.email}`">✉ {{ company.email }}</a>
        <span class="topbar__sep" aria-hidden="true">·</span>
        <span class="topbar__hours">🕘 {{ company.hours }}</span>
      </div>
    </div>
  </div>

  <!-- main bar — sticky, constant height -->
  <header class="nav">
    <div class="contenedor nav__inner">
      <a href="#top" class="nav__logo" @click="open = false">
        <img class="nav__logo-mark" src="/logo-mark.png" :alt="company.name" />
        <span class="nav__logo-wm">
          <span class="nav__logo-name">{{ company.logoText }}</span>
          <span class="nav__logo-services">Services</span>
        </span>
      </a>

      <nav class="nav__links" :class="{ 'nav__links--open': open }">
        <a
          v-for="l in links"
          :key="l.to"
          :href="l.to"
          class="nav__link"
          :class="{ 'is-active': activeId === l.to.slice(1) }"
          @click="open = false"
        >
          {{ l.name }}
        </a>
        <a :href="`tel:${company.phoneLink}`" class="nav__phone">
          <span class="nav__phone-ico">📞</span>{{ company.phone }}
        </a>
        <a href="#quote" class="btn btn--verde nav__cta" @click="open = false">
          Free Quote
        </a>
      </nav>

      <button class="nav__toggle" @click="open = !open" aria-label="Menu">
        <span :class="{ open }"></span>
        <span :class="{ open }"></span>
        <span :class="{ open }"></span>
      </button>
    </div>
  </header>
</template>

<style scoped>
/* ---------- slim utility bar (scrolls away) ---------- */
.topbar {
  background: linear-gradient(90deg, var(--color-dark-2), var(--color-dark));
  border-bottom: 1px solid var(--color-dark-borde);
  font-size: 0.78rem;
  color: var(--color-dark-texto);
}
.topbar__inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 38px;
  gap: 18px;
}
.topbar__tag { font-weight: 600; letter-spacing: 0.02em; color: #cfe0ea; white-space: nowrap; }
.topbar__right { display: flex; align-items: center; gap: 12px; white-space: nowrap; }
.topbar__right a { color: var(--color-dark-texto); transition: color var(--transicion); }
.topbar__right a:hover { color: var(--color-acento); }
.topbar__sep { opacity: 0.5; }

/* ---------- main bar (sticky, constant height) ---------- */
.nav {
  position: sticky;
  top: 0;
  z-index: 50;
  background: rgba(11, 27, 36, 0.9);
  backdrop-filter: blur(14px);
  border-bottom: 1px solid var(--color-dark-borde);
  box-shadow: 0 6px 24px rgba(0, 0, 0, 0.25);
}
.nav__inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 72px;
}

/* ---------- logo (vertical lockup) ---------- */
.nav__logo { display: flex; flex-direction: column; align-items: center; gap: 3px; line-height: 1; }
.nav__logo-mark { display: block; height: 32px; width: auto; }
.nav__logo-wm { display: flex; flex-direction: column; align-items: center; gap: 2px; line-height: 1; }
.nav__logo-name {
  font-family: var(--fuente-display);
  text-transform: uppercase;
  letter-spacing: 0.12em;
  font-weight: 700;
  font-size: 1.05rem;
  color: #fff;
}
.nav__logo-services {
  display: flex;
  align-items: center;
  gap: 7px;
  font-family: var(--fuente-display);
  text-transform: uppercase;
  letter-spacing: 0.28em;
  font-size: 0.48rem;
  font-weight: 600;
  color: var(--color-acento);
  padding-left: 0.28em;
}
.nav__logo-services::before,
.nav__logo-services::after {
  content: '';
  height: 1px;
  flex: 1;
  max-width: 20px;
  background: var(--color-acento);
  opacity: 0.8;
}

/* ---------- links ---------- */
.nav__links { display: flex; align-items: center; gap: 30px; }
.nav__link {
  font-family: var(--fuente-display);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  font-size: 0.9rem;
  font-weight: 500;
  color: var(--color-dark-texto);
  transition: color var(--transicion);
  position: relative;
  padding: 6px 0;
}
.nav__link::after {
  content: '';
  position: absolute;
  left: 0;
  bottom: 0;
  width: 100%;
  height: 2px;
  border-radius: 2px;
  background: linear-gradient(90deg, var(--color-acento), var(--color-primario));
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 0.28s ease;
}
.nav__link:hover { color: #fff; }
.nav__link:hover::after,
.nav__link.is-active::after { transform: scaleX(1); }
.nav__link.is-active { color: #fff; }

.nav__phone {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  font-weight: 600;
  color: #fff;
  font-size: 0.95rem;
  padding: 8px 16px;
  border: 1px solid var(--color-dark-borde);
  border-radius: 999px;
  transition: var(--transicion);
}
.nav__phone:hover { border-color: var(--color-acento); background: rgba(89, 173, 71, 0.12); }
.nav__phone-ico { font-size: 0.85rem; }
.nav__cta { padding: 11px 24px; box-shadow: 0 8px 20px rgba(89, 173, 71, 0.28); }

/* ---------- mobile toggle ---------- */
.nav__toggle {
  display: none;
  flex-direction: column;
  gap: 5px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 6px;
}
.nav__toggle span {
  width: 24px;
  height: 2px;
  background: #fff;
  border-radius: 2px;
  transition: var(--transicion);
}
.nav__toggle span.open:nth-child(1) { transform: translateY(7px) rotate(45deg); }
.nav__toggle span.open:nth-child(2) { opacity: 0; }
.nav__toggle span.open:nth-child(3) { transform: translateY(-7px) rotate(-45deg); }

@media (max-width: 1040px) {
  .topbar__hours { display: none; }
}
@media (max-width: 900px) {
  .topbar { display: none; }
  .nav__toggle { display: flex; }
  .nav__links {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    flex-direction: column;
    gap: 20px;
    background: var(--color-dark);
    padding: 32px 24px;
    border-bottom: 1px solid var(--color-dark-borde);
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
    transform: translateY(-150%);
    opacity: 0;
    pointer-events: none;
    transition: var(--transicion);
  }
  .nav__links--open {
    transform: translateY(0);
    opacity: 1;
    pointer-events: auto;
  }
  .nav__link::after { display: none; }
  .nav__phone { border: none; padding: 0; }
  .nav__cta { width: 100%; }
}
</style>
