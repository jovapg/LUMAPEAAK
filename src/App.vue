<script setup>
import NavBar from './components/NavBar.vue'
import AppFooter from './components/AppFooter.vue'
import content from './data/content.json'
const { company } = content
const smsHref = `sms:${company.phoneLink}?body=${encodeURIComponent("Hi LumaPeak! I'd like to ask about your services.")}`
</script>

<template>
  <div class="shell">
    <a href="#main" class="skip-link">Skip to content</a>
    <NavBar />
    <main id="main" tabindex="-1">
      <router-view v-slot="{ Component }">
        <transition name="page" mode="out-in">
          <component :is="Component" />
        </transition>
      </router-view>
    </main>
    <AppFooter />
  </div>

  <!-- Sticky mobile call-to-action bar -->
  <div class="mobile-cta">
    <a :href="`tel:${company.phoneLink}`" class="mobile-cta__btn mobile-cta__call">📞 Call Now</a>
    <a href="#quote" class="mobile-cta__btn mobile-cta__quote">Free Quote</a>
  </div>

  <!-- Floating SMS button (US customers prefer texting) -->
  <a :href="smsHref" class="sms-fab" aria-label="Text us">
    <span class="sms-fab__icon" aria-hidden="true">
      <svg viewBox="0 0 24 24" width="26" height="26" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <path d="M21 11.5a8.38 8.38 0 0 1-8.5 8.5 8.5 8.5 0 0 1-3.9-.9L3 21l1.9-5.6a8.5 8.5 0 0 1-.9-3.9A8.38 8.38 0 0 1 12.5 3 8.38 8.38 0 0 1 21 11.5z" />
      </svg>
    </span>
    <span class="sms-fab__label">Text us</span>
  </a>
</template>

<style>
.page-enter-active,
.page-leave-active {
  transition: opacity 0.25s ease, transform 0.25s ease;
}
.page-enter-from {
  opacity: 0;
  transform: translateY(12px);
}
.page-leave-to {
  opacity: 0;
}

/* ---------- Floating SMS button ---------- */
.sms-fab {
  position: fixed;
  left: 22px;
  top: 80%;
  z-index: 1000;
  display: flex;
  align-items: center;
  height: 58px;
  width: 58px;
  padding: 0;
  border-radius: 50px;
  background: var(--color-acento);
  color: #fff;
  overflow: hidden;
  white-space: nowrap;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.4);
  transition: width 0.3s ease, background 0.2s ease, box-shadow 0.2s ease;
  animation: sms-pulse 2.6s ease-in-out infinite;
}
.sms-fab__icon {
  flex: 0 0 58px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.sms-fab__label {
  font-family: var(--fuente-display);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  font-weight: 600;
  font-size: 0.9rem;
  opacity: 0;
  transition: opacity 0.2s ease;
}
.sms-fab:hover {
  width: 158px;
  background: var(--color-acento-hover);
  color: #fff;
  animation: none;
}
.sms-fab:hover .sms-fab__label { opacity: 1; }

@keyframes sms-pulse {
  0%, 100% { box-shadow: 0 12px 30px rgba(0, 0, 0, 0.4), 0 0 0 0 rgba(89, 173, 71, 0.5); }
  50% { box-shadow: 0 12px 30px rgba(0, 0, 0, 0.4), 0 0 0 12px rgba(89, 173, 71, 0); }
}

/* ---------- Sticky mobile CTA bar ---------- */
.mobile-cta { display: none; }
@media (max-width: 768px) {
  .mobile-cta {
    display: flex;
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 1500;
    gap: 10px;
    padding: 10px 12px calc(10px + env(safe-area-inset-bottom));
    background: rgba(10, 24, 32, 0.96);
    backdrop-filter: blur(10px);
    border-top: 1px solid var(--color-dark-borde);
  }
  .mobile-cta__btn {
    flex: 1;
    text-align: center;
    padding: 13px 10px;
    border-radius: 10px;
    font-family: var(--fuente-display);
    text-transform: uppercase;
    letter-spacing: 0.04em;
    font-weight: 600;
    font-size: 0.9rem;
  }
  .mobile-cta__call {
    background: rgba(255, 255, 255, 0.1);
    color: #fff;
    border: 1px solid var(--color-dark-borde);
  }
  .mobile-cta__quote {
    background: var(--color-acento);
    color: #fff;
    box-shadow: 0 8px 20px rgba(89, 173, 71, 0.3);
  }
}

@media (max-width: 600px) {
  .sms-fab { left: 14px; top: auto; bottom: 84px; height: 52px; width: 52px; }
  .sms-fab__icon { flex-basis: 52px; }
  .sms-fab:hover { width: 52px; }
  .sms-fab:hover .sms-fab__label { opacity: 0; }
}
@media (prefers-reduced-motion: reduce) {
  .sms-fab { animation: none; }
}
</style>
