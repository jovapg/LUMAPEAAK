<script setup>
import { ref, computed, watch, onBeforeUnmount } from 'vue'
import content from '../data/content.json'
const { company, services, legal } = content
const year = 2026

const email = ref('')
const subscribed = ref(false)
function subscribe() {
  if (!email.value) return
  const subject = 'Newsletter signup'
  const body = `Please add me to your list: ${email.value}`
  window.location.href =
    `mailto:${company.email}?subject=${encodeURIComponent(subject)}&body=${encodeURIComponent(body)}`
  subscribed.value = true
}

/* ---- Legal modals ---- */
const openModal = ref(null) // 'terms' | 'privacy' | null
const current = computed(() => (openModal.value ? legal[openModal.value] : null))
function closeModal() { openModal.value = null }
function onKey(e) { if (e.key === 'Escape') closeModal() }
watch(openModal, (val) => {
  if (val) window.addEventListener('keydown', onKey)
  else window.removeEventListener('keydown', onKey)
})
onBeforeUnmount(() => window.removeEventListener('keydown', onKey))
</script>

<template>
  <footer class="footer">
    <div class="contenedor footer__grid">
      <div class="footer__brand">
        <img class="footer__logo-img" src="/logo-full.png" :alt="company.name" />
        <p class="footer__slogan">{{ company.slogan }}</p>
        <div class="footer__socials">
          <a v-for="s in company.socials" :key="s.name" :href="s.url" target="_blank" rel="noopener">
            {{ s.name }}
          </a>
        </div>
      </div>

      <div class="footer__col">
        <h4>Services</h4>
        <ul>
          <li v-for="s in services.slice(0, 6)" :key="s.title">
            <a href="#services">{{ s.title }}</a>
          </li>
        </ul>
      </div>

      <div class="footer__col">
        <h4>Company</h4>
        <ul>
          <li><a href="#top">Home</a></li>
          <li><a href="#services">Services</a></li>
          <li><a href="#service-areas">Service Areas</a></li>
          <li><a href="#about">About</a></li>
          <li><a href="#quote">Get a Quote</a></li>
        </ul>
      </div>

      <div class="footer__col footer__news">
        <h4>Stay in the loop</h4>
        <p class="footer__news-text">
          Seasonal tips and offers for keeping your home looking its best.
        </p>
        <form class="footer__sub" @submit.prevent="subscribe">
          <input v-model="email" type="email" required placeholder="Your email" aria-label="Your email" />
          <button type="submit" class="btn btn--verde">Subscribe</button>
        </form>
        <p v-if="subscribed" class="footer__sub-ok">✓ Thanks! Your email app should open to confirm.</p>
        <ul class="footer__contact">
          <li><a :href="`tel:${company.phoneLink}`">📞 {{ company.phone }}</a></li>
          <li><a :href="`mailto:${company.email}`">✉ {{ company.email }}</a></li>
          <li>🕘 {{ company.hours }}</li>
        </ul>
      </div>
    </div>

    <div class="footer__base">
      <div class="contenedor footer__base-inner">
        <span>© {{ year }} {{ company.name }}. All rights reserved.</span>
        <div class="footer__legal">
          <button type="button" @click="openModal = 'terms'">Terms &amp; Conditions</button>
          <span aria-hidden="true">·</span>
          <button type="button" @click="openModal = 'privacy'">Privacy Policy</button>
        </div>
        <span class="footer__tag">Licensed &amp; Insured · Residential Only</span>
      </div>
    </div>
  </footer>

  <!-- Legal modal -->
  <Teleport to="body">
    <div v-if="current" class="legal-overlay" @click.self="closeModal">
      <div class="legal-modal" role="dialog" aria-modal="true" :aria-label="current.title">
        <header class="legal-modal__head">
          <h3>{{ current.title }}</h3>
          <button type="button" class="legal-modal__x" aria-label="Close" @click="closeModal">×</button>
        </header>
        <div class="legal-modal__body">
          <p class="legal-modal__eff">{{ current.effective }}</p>
          <section v-for="s in current.sections" :key="s.heading" class="legal-modal__section">
            <h4>{{ s.heading }}</h4>
            <p v-for="(p, i) in s.body" :key="i">{{ p }}</p>
          </section>
        </div>
        <footer class="legal-modal__foot">
          <button type="button" class="btn btn--verde" @click="closeModal">Close</button>
        </footer>
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
.footer {
  background: var(--color-dark);
  color: var(--color-dark-texto);
  padding-top: 72px;
  border-top: 1px solid var(--color-dark-borde);
  border-bottom-left-radius: var(--frame-radio);
  border-bottom-right-radius: var(--frame-radio);
}
.footer__grid {
  display: grid;
  grid-template-columns: 1.4fr 1fr 1fr 1.4fr;
  gap: 48px;
  padding-bottom: 48px;
}
.footer__logo-img {
  display: block;
  width: 200px;
  height: auto;
  margin: 0 0 14px;
}
.footer__slogan { max-width: 300px; font-size: 0.95rem; font-style: italic; }
.footer__socials { display: flex; gap: 18px; margin-top: 20px; }
.footer__socials a {
  font-size: 0.9rem;
  color: #8fa9b8;
  transition: color var(--transicion);
}
.footer__socials a:hover { color: var(--color-acento); }

.footer__col h4 {
  color: #fff;
  font-size: 0.95rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 18px;
}
.footer__col ul li { margin-bottom: 12px; font-size: 0.92rem; }
.footer__col ul li a { color: var(--color-dark-texto); transition: color var(--transicion); }
.footer__col ul li a:hover { color: #fff; }

/* newsletter */
.footer__news-text { font-size: 0.92rem; margin-bottom: 16px; max-width: 320px; }
.footer__sub { display: flex; gap: 10px; max-width: 360px; }
.footer__sub input {
  flex: 1;
  min-width: 0;
  font-family: inherit;
  font-size: 0.95rem;
  padding: 12px 14px;
  border-radius: 8px;
  border: 1px solid var(--color-dark-borde);
  background: rgba(255, 255, 255, 0.05);
  color: #fff;
  transition: border-color var(--transicion);
}
.footer__sub input::placeholder { color: #7e98a6; }
.footer__sub input:focus { outline: none; border-color: var(--color-acento); }
.footer__sub .btn { padding: 12px 20px; white-space: nowrap; }
.footer__sub-ok { color: var(--color-acento); font-size: 0.86rem; margin-top: 10px; font-weight: 600; }
.footer__contact { margin-top: 22px; display: grid; gap: 10px; }
.footer__contact li { font-size: 0.92rem; }
.footer__contact a { color: var(--color-dark-texto); transition: color var(--transicion); }
.footer__contact a:hover { color: #fff; }

.footer__base {
  border-top: 1px solid var(--color-dark-borde);
  padding: 24px 0;
  font-size: 0.86rem;
  color: #7e98a6;
}
.footer__base-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 8px 18px;
}
.footer__legal {
  display: flex;
  align-items: center;
  gap: 10px;
}
.footer__legal button {
  background: none;
  border: none;
  cursor: pointer;
  color: var(--color-dark-texto);
  font: inherit;
  padding: 0;
  transition: color var(--transicion);
}
.footer__legal button:hover { color: var(--color-acento); text-decoration: underline; }

/* ---------- Legal modal ---------- */
.legal-overlay {
  position: fixed;
  inset: 0;
  z-index: 2000;
  background: rgba(4, 12, 18, 0.72);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  animation: legal-fade 0.2s ease;
}
.legal-modal {
  background: #fff;
  color: var(--color-texto);
  width: 100%;
  max-width: 640px;
  max-height: 86vh;
  border-radius: var(--radio);
  box-shadow: 0 30px 80px rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  animation: legal-pop 0.25s ease;
}
.legal-modal__head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 22px 26px;
  border-bottom: 1px solid var(--color-borde);
}
.legal-modal__head h3 {
  font-family: var(--fuente-display);
  text-transform: uppercase;
  font-size: 1.3rem;
  color: var(--color-texto);
}
.legal-modal__x {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1.9rem;
  line-height: 1;
  color: var(--color-texto-suave);
  width: 36px;
  height: 36px;
  border-radius: 8px;
  flex-shrink: 0;
  transition: var(--transicion);
}
.legal-modal__x:hover { background: var(--color-fondo-alt); color: var(--color-texto); }
.legal-modal__body {
  padding: 22px 26px;
  overflow-y: auto;
}
.legal-modal__eff {
  font-size: 0.82rem;
  color: var(--color-texto-suave);
  font-style: italic;
  margin-bottom: 18px;
}
.legal-modal__section { margin-bottom: 18px; }
.legal-modal__section h4 {
  font-size: 0.98rem;
  color: var(--color-primario);
  margin-bottom: 6px;
}
.legal-modal__section p {
  font-size: 0.92rem;
  color: var(--color-texto-suave);
  line-height: 1.6;
}
.legal-modal__foot {
  padding: 16px 26px;
  border-top: 1px solid var(--color-borde);
  display: flex;
  justify-content: flex-end;
}
@keyframes legal-fade { from { opacity: 0; } to { opacity: 1; } }
@keyframes legal-pop { from { opacity: 0; transform: translateY(14px) scale(0.98); } to { opacity: 1; transform: none; } }

@media (max-width: 900px) {
  .footer__grid { grid-template-columns: 1fr 1fr; gap: 40px; }
}
@media (max-width: 560px) {
  .footer__grid { grid-template-columns: 1fr; gap: 32px; }
  .footer__sub { max-width: 100%; }
}
</style>
