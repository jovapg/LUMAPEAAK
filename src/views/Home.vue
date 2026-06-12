<script setup>
import { reactive, ref, computed, watch, onMounted, onBeforeUnmount } from 'vue'
import VideoShowcase from '../components/VideoShowcase.vue'
import ServiceIcon from '../components/ServiceIcon.vue'
import ServiceAreaMap from '../components/ServiceAreaMap.vue'
import ReviewsCarousel from '../components/ReviewsCarousel.vue'
import content from '../data/content.json'
const { home, company, services, experience, areas, about, trust, quote, faq } = content

/* ---- Hero rotating circle ---- */
const heroImages = home.heroImages || []
const heroIndex = ref(0)
let heroTimer

/* ---- Service selector + auto gallery ---- */
const selectedIndex = ref(0)
const selected = computed(() => services[selectedIndex.value])
const marqueePaused = ref(false)
// duplicate list so the marquee can loop seamlessly
const marqueeServices = computed(() => [...services, ...services])
function selectService(i) {
  selectedIndex.value = i
  galleryIndex.value = 0
}

const galleryIndex = ref(0)
let galleryTimer
watch(selectedIndex, () => { galleryIndex.value = 0 })

const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
onMounted(() => {
  if (reduceMotion) return // respect users who prefer no auto-animation
  if (heroImages.length > 1) {
    heroTimer = setInterval(() => {
      heroIndex.value = (heroIndex.value + 1) % heroImages.length
    }, 4500)
  }
  galleryTimer = setInterval(() => {
    const imgs = selected.value.images || []
    if (imgs.length > 1) galleryIndex.value = (galleryIndex.value + 1) % imgs.length
  }, 3500)
})
onBeforeUnmount(() => {
  clearInterval(heroTimer)
  clearInterval(galleryTimer)
})

/* ---- Quote form ---- */
const form = reactive({ name: '', phone: '', email: '', address: '', service: '', notes: '' })
const sent = ref(false)
const sending = ref(false)
const sendError = ref('')

function mailtoFallback() {
  const subject = `Quote Request — ${form.service || 'General'} — ${form.name}`
  const body =
    `Full Name: ${form.name}\n` +
    `Phone: ${form.phone}\n` +
    `Email: ${form.email}\n` +
    `Property Address: ${form.address}\n` +
    `Service Requested: ${form.service}\n` +
    `Additional Notes: ${form.notes || '—'}`
  window.location.href =
    `mailto:${company.email}?subject=${encodeURIComponent(subject)}&body=${encodeURIComponent(body)}`
}

async function submitQuote() {
  sendError.value = ''

  // No Web3Forms key configured yet -> fall back to opening the user's mail app
  if (!quote.web3formsKey) {
    mailtoFallback()
    sent.value = true
    return
  }

  sending.value = true
  try {
    const res = await fetch('https://api.web3forms.com/submit', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', Accept: 'application/json' },
      body: JSON.stringify({
        access_key: quote.web3formsKey,
        subject: `New Quote Request — ${form.service || 'General'} — ${form.name}`,
        from_name: 'LumaPeak Website',
        replyto: form.email,
        'Full Name': form.name,
        Phone: form.phone,
        Email: form.email,
        'Property Address': form.address,
        'Service Requested': form.service,
        'Additional Notes': form.notes || '—',
      }),
    })
    const data = await res.json()
    if (data.success) {
      sent.value = true
      Object.keys(form).forEach((k) => (form[k] = ''))
    } else {
      sendError.value = 'Something went wrong. Please call or email us directly.'
    }
  } catch (e) {
    sendError.value = 'Network error. Please call or email us directly.'
  } finally {
    sending.value = false
  }
}
</script>

<template>
  <!-- ===== HERO ===== -->
  <section id="top" class="hero">
    <!-- 🖼️ Swap the gradient for a real photo: background: url('/hero.jpg') center/cover; -->
    <div class="hero__bg"></div>
    <div class="hero__overlay"></div>

    <div class="contenedor hero__inner fade-in">
      <div class="hero__text">
        <span class="eyebrow hero__eyebrow">{{ company.slogan }}</span>
        <h1 class="hero__title">
          Exterior cleaning<br />
          <span class="hero__accent">done right.</span>
        </h1>
        <p class="hero__subtitle">{{ home.heroSubtitle }}</p>
        <div class="hero__actions">
          <a href="#quote" class="btn btn--verde">{{ home.ctaPrimary }}</a>
          <a :href="`tel:${company.phoneLink}`" class="btn btn--light">{{ home.ctaSecondary }}</a>
        </div>
        <div class="hero__trust">
          <span>★ Licensed &amp; Insured</span>
          <span>✓ Same-Day Response</span>
          <span>✓ 100% Satisfaction</span>
        </div>
      </div>

      <div class="hero__media">
        <div class="hero__ring"></div>
        <div class="hero__circle">
          <img
            v-for="(img, i) in heroImages"
            :key="img"
            :src="img"
            class="hero__photo"
            :class="{ 'is-active': i === heroIndex }"
            alt="LumaPeak team cleaning"
          />
        </div>
        <div class="hero__badge">
          <span>★</span>
          <small>Licensed<br />&amp; Insured</small>
        </div>
      </div>
    </div>
  </section>

  <!-- ===== SERVICES (interactive selector) ===== -->
  <section id="services" class="services-x seccion--dark">
    <!-- auto-scrolling card marquee (overlaps the hero) -->
    <div
      class="svc-marquee"
      @mouseenter="marqueePaused = true"
      @mouseleave="marqueePaused = false"
    >
      <div class="svc-marquee__track" :class="{ 'is-paused': marqueePaused }">
        <button
          v-for="(s, i) in marqueeServices"
          :key="i"
          type="button"
          class="svc-chip"
          :class="{ 'is-active': (i % services.length) === selectedIndex }"
          @click="selectService(i % services.length)"
        >
          <span class="svc-chip__icon"><ServiceIcon :name="s.iconKey" /></span>
          <span class="svc-chip__title">{{ s.title }}</span>
          <span class="svc-chip__price">{{ s.price }}</span>
        </button>
      </div>
    </div>

    <!-- white detail panel for the selected service -->
    <div class="contenedor">
      <div class="svc-head">
        <span class="eyebrow">What We Do</span>
        <h2 class="svc-head__title">Our Services</h2>
        <p class="svc-head__sub">Tap any service to see the details, what's included and real photos of our work.</p>
      </div>
      <div class="svc-detail">
        <div class="svc-detail__gallery">
          <template v-if="selected.images && selected.images.length">
            <img
              v-for="(img, gi) in selected.images"
              :key="img"
              :src="img"
              class="svc-detail__photo"
              :class="{ 'is-active': gi === galleryIndex }"
              :alt="selected.title"
              loading="lazy"
              decoding="async"
            />
          </template>
          <div v-else class="svc-detail__empty"><ServiceIcon :name="selected.iconKey" /></div>
          <span class="svc-detail__badge"><ServiceIcon :name="selected.iconKey" /></span>
          <div v-if="selected.images && selected.images.length > 1" class="svc-detail__dots">
            <button
              v-for="(img, gi) in selected.images"
              :key="gi"
              type="button"
              class="svc-detail__dot"
              :class="{ 'is-active': gi === galleryIndex }"
              :aria-label="`Photo ${gi + 1}`"
              @click="galleryIndex = gi"
            ></button>
          </div>
        </div>

        <div class="svc-detail__info">
          <span class="svc-detail__price">{{ selected.price }}</span>
          <h3 class="svc-detail__title">{{ selected.title }}</h3>
          <p class="svc-detail__desc">{{ selected.description }}</p>
          <ul class="service__list svc-detail__list">
            <li v-for="item in selected.includes" :key="item">{{ item }}</li>
          </ul>
          <a href="#quote" class="btn btn--verde svc-detail__cta">Get a Free Quote</a>
        </div>
      </div>
    </div>
  </section>

  <!-- ===== VIDEO SHOWCASE ===== -->
  <VideoShowcase />

  <!-- ===== SERVICE AREAS ===== -->
  <section id="service-areas" class="seccion seccion--dark">
    <div class="contenedor">
      <div class="center-head">
        <span class="eyebrow">Where We Work</span>
        <h2 class="titulo-seccion">{{ areas.title }}</h2>
        <p class="subtitulo-seccion">{{ areas.subtitle }}</p>
      </div>
    </div>
    <ServiceAreaMap />
  </section>

  <!-- ===== ABOUT + TRUST ===== -->
  <section id="about" class="seccion seccion--dark alt-shade about-sec">
    <span class="about-glow about-glow--1" aria-hidden="true"></span>
    <span class="about-glow about-glow--2" aria-hidden="true"></span>
    <div class="contenedor about2">
      <div class="about2__info">
        <span class="eyebrow">About Us</span>
        <h2 class="about2__title">{{ about.title }}</h2>
        <p class="about2__lead">{{ about.intro }}</p>
        <p class="about2__body">{{ about.body }}</p>
        <ul class="about2__chips">
          <li>Licensed &amp; Insured</li>
          <li>Same-Day Response</li>
          <li>Residential Specialists</li>
        </ul>
        <div class="about2__actions">
          <a href="#quote" class="btn btn--verde">Get a Free Quote</a>
          <a :href="`tel:${company.phoneLink}`" class="btn btn--light">📞 Call Us</a>
        </div>
      </div>
      <div class="about2__cards">
        <div v-for="(t, i) in trust.items" :key="t.title" class="tcard">
          <span class="tcard__num">{{ String(i + 1).padStart(2, '0') }}</span>
          <div class="tcard__icon">{{ t.icon }}</div>
          <div class="tcard__text">
            <h4>{{ t.title }}</h4>
            <p>{{ t.text }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- experience band, merged into the About block -->
    <div class="contenedor exp__grid about-exp">
      <div class="exp__text">
        <span class="eyebrow">{{ experience.eyebrow }}</span>
        <h2 class="exp__title">{{ experience.title }}</h2>
        <p class="exp__body">{{ experience.body }}</p>
        <ul class="exp__list">
          <li v-for="p in experience.points" :key="p">{{ p }}</li>
        </ul>
        <a href="#services" class="btn btn--verde">See Our Services</a>
      </div>
      <div class="exp__img">
        <img src="/hero/hero-2.jpg" :alt="`${company.name} team at work`" loading="lazy" decoding="async" />
      </div>
    </div>
  </section>

  <!-- ===== REVIEWS ===== -->
  <ReviewsCarousel />

  <!-- ===== FAQ ===== -->
  <section id="faq" class="seccion seccion--dark faq-sec">
    <div class="contenedor faq">
      <div class="center-head">
        <span class="eyebrow">{{ faq.eyebrow }}</span>
        <h2 class="titulo-seccion">{{ faq.title }}</h2>
        <p class="subtitulo-seccion">{{ faq.subtitle }}</p>
      </div>
      <ul class="faq__list">
        <li v-for="(item, i) in faq.items" :key="i" class="faq__item">
          <details class="faq__details">
            <summary class="faq__q">
              <span>{{ item.q }}</span>
              <span class="faq__icon" aria-hidden="true"></span>
            </summary>
            <p class="faq__a">{{ item.a }}</p>
          </details>
        </li>
      </ul>
    </div>
  </section>

  <!-- ===== QUOTE FORM ===== -->
  <section id="quote" class="seccion seccion--dark quote-sec">
    <span class="quote-glow quote-glow--1" aria-hidden="true"></span>
    <span class="quote-glow quote-glow--2" aria-hidden="true"></span>
    <div class="contenedor quote">
      <div class="quote__info">
        <span class="eyebrow">Get Started</span>
        <h2 class="quote__title">{{ quote.title }}</h2>
        <p class="quote__sub">{{ quote.subtitle }}</p>

        <ul class="quote__perks">
          <li><span class="quote__perk-ico">✓</span> Free, no-obligation estimate</li>
          <li><span class="quote__perk-ico">✓</span> Same-day response — we reply fast</li>
          <li><span class="quote__perk-ico">✓</span> Licensed &amp; insured professionals</li>
        </ul>

        <div class="quote__contact">
          <a class="qcontact" :href="`tel:${company.phoneLink}`">
            <span class="qcontact__ico">📞</span>
            <span class="qcontact__txt"><small>Call us</small><strong>{{ company.phone }}</strong></span>
          </a>
          <a class="qcontact" :href="`mailto:${company.email}`">
            <span class="qcontact__ico">✉</span>
            <span class="qcontact__txt"><small>Email</small><strong>{{ company.email }}</strong></span>
          </a>
          <div class="qcontact">
            <span class="qcontact__ico">🕘</span>
            <span class="qcontact__txt"><small>Hours</small><strong>{{ company.hours }}</strong></span>
          </div>
        </div>
      </div>

      <form class="quote__form" @submit.prevent="submitQuote">
        <div class="quote__form-head">
          <h3>Get your free quote</h3>
          <p><span class="quote__bolt">⚡</span> Takes under a minute — no obligation.</p>
        </div>

        <div class="quote__row">
          <label>Full Name <span class="req">*</span>
            <input v-model="form.name" type="text" required placeholder="John Smith" />
          </label>
          <label>Phone Number <span class="req">*</span>
            <input v-model="form.phone" type="tel" required placeholder="(425) 000-0000" />
          </label>
        </div>
        <div class="quote__row">
          <label>Email Address <span class="req">*</span>
            <input v-model="form.email" type="email" required placeholder="you@email.com" />
          </label>
          <label>Property Address <span class="req">*</span>
            <input v-model="form.address" type="text" required placeholder="123 Main St, Seattle, WA" />
          </label>
        </div>
        <label>Service Requested <span class="req">*</span>
          <select v-model="form.service" required :class="{ 'is-placeholder': !form.service }">
            <option value="" disabled>Select a service…</option>
            <option v-for="opt in quote.serviceOptions" :key="opt" :value="opt">{{ opt }}</option>
          </select>
        </label>
        <label>Additional Notes <span class="opt">(optional)</span>
          <textarea v-model="form.notes" rows="4" placeholder="Tell us anything else about your project…"></textarea>
        </label>
        <button type="submit" class="btn btn--verde quote__submit" :disabled="sending">
          <template v-if="sending">Sending…</template>
          <template v-else>
            Request a Free Estimate
            <span class="quote__submit-arrow" aria-hidden="true">→</span>
          </template>
        </button>
        <p class="quote__reassure">🔒 No spam — your details are only used to prepare your quote.</p>
        <p v-if="sent" class="quote__ok">
          ✓ Thank you! Your request has been sent — we typically respond the same day.
        </p>
        <p v-if="sendError" class="quote__err">{{ sendError }}</p>
      </form>
    </div>
  </section>
</template>

<style scoped>
.center-head { max-width: 640px; margin: 0 auto; text-align: center; }
.center-head .subtitulo-seccion { margin-bottom: 0; }
.center-head .eyebrow { margin-bottom: 12px; }
.center-head .titulo-seccion + .subtitulo-seccion { margin-top: 14px; }
section .center-head { margin-bottom: 56px; }

/* tighter spacing around the service-areas map */
#service-areas { padding-top: 48px; padding-bottom: 56px; }
#service-areas .center-head { margin-bottom: 28px; }

/* ---------- HERO ---------- */
.hero {
  position: relative;
  display: flex;
  align-items: center;
  overflow: hidden;
  padding: 16px 0 90px;
}
.hero__bg {
  position: absolute;
  inset: 0;
  background:
    radial-gradient(circle at 75% 35%, rgba(0, 118, 178, 0.30), transparent 55%),
    radial-gradient(circle at 15% 85%, rgba(89, 173, 71, 0.20), transparent 50%),
    linear-gradient(135deg, #0e2733 0%, #06141c 100%);
}
.hero__overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, rgba(6, 20, 28, 0.82) 0%, rgba(6, 20, 28, 0.25) 60%, rgba(6, 20, 28, 0.5) 100%);
}
.hero__inner {
  position: relative;
  z-index: 2;
  color: #fff;
  display: grid;
  grid-template-columns: 1.05fr 0.95fr;
  align-items: center;
  gap: 56px;
  width: 100%;
}
.hero__text { max-width: 620px; }
.hero__eyebrow { color: var(--color-acento); }
.hero__title {
  font-family: var(--fuente-display);
  text-transform: uppercase;
  font-weight: 700;
  font-size: clamp(2.8rem, 6vw, 5rem);
  line-height: 0.98;
  margin-bottom: 24px;
}
.hero__accent { color: var(--color-acento); }
.hero__subtitle {
  font-size: 1.2rem;
  color: var(--color-dark-texto);
  max-width: 560px;
  margin-bottom: 34px;
}
.hero__actions { display: flex; gap: 16px; flex-wrap: wrap; margin-bottom: 34px; }
.hero__trust {
  display: flex;
  flex-wrap: wrap;
  gap: 10px 22px;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--color-dark-texto);
  margin-bottom: 76px;
}
.hero__trust span { white-space: nowrap; }

/* circular rotating media */
.hero__media {
  position: relative;
  justify-self: center;
  width: min(581px, 90vw);
  aspect-ratio: 1;
}
.hero__ring {
  position: absolute;
  inset: -18px;
  border-radius: 50%;
  border: 2px solid rgba(89, 173, 71, 0.45);
  border-top-color: var(--color-primario);
  border-right-color: var(--color-acento);
}
.hero__circle {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  overflow: hidden;
  box-shadow: 0 30px 70px rgba(0, 0, 0, 0.5);
  background: #0a2230;
}
.hero__photo {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  transform: scale(1.06);
  transition: opacity 1.1s ease, transform 5s ease;
}
.hero__photo.is-active { opacity: 1; transform: scale(1); }
.hero__badge {
  position: absolute;
  bottom: 78px;
  left: -8px;
  z-index: 12;
  width: 104px;
  height: 104px;
  border-radius: 50%;
  background: var(--color-acento);
  color: #fff;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  gap: 2px;
  box-shadow: var(--sombra-hover);
  border: 4px solid #0a1822;
}
.hero__badge span { font-size: 1.3rem; }
.hero__badge small { font-size: 0.62rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.05em; line-height: 1.2; }

/* ---------- SERVICES (interactive selector) ---------- */
.services-x { overflow: visible; padding-bottom: 88px; }

/* compact heading above the white detail panel */
.svc-head { text-align: center; max-width: 640px; margin: 0 auto 22px; }
.svc-head .eyebrow { margin-bottom: 8px; }
.svc-head__title {
  font-family: var(--fuente-display);
  text-transform: uppercase;
  font-size: clamp(1.4rem, 3vw, 2.1rem);
  font-weight: 700;
  line-height: 1.06;
  color: #fff;
  margin-bottom: 6px;
}
.svc-head__sub { color: var(--color-dark-texto); font-size: 0.92rem; }

/* auto-scrolling marquee — pulled up to overlap the hero circle */
.svc-marquee {
  position: relative;
  z-index: 10;
  margin: -150px 0 4px;
  overflow: hidden;
  padding: 6px 0;
  -webkit-mask-image: linear-gradient(90deg, transparent, #000 6%, #000 94%, transparent);
  mask-image: linear-gradient(90deg, transparent, #000 6%, #000 94%, transparent);
}
.svc-marquee__track {
  display: flex;
  gap: 18px;
  width: max-content;
  padding: 6px 0;
  animation: svc-scroll 42s linear infinite;
}
.svc-marquee__track.is-paused { animation-play-state: paused; }
@keyframes svc-scroll { from { transform: translateX(0); } to { transform: translateX(-50%); } }
.svc-chip {
  flex: 0 0 auto;
  width: 272px;
  text-align: left;
  cursor: pointer;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid var(--color-dark-borde);
  border-radius: 16px;
  padding: 30px 28px;
  color: #fff;
  display: flex;
  flex-direction: column;
  gap: 12px;
  transition: border-color var(--transicion), background var(--transicion), transform var(--transicion);
}
.svc-chip:hover { transform: translateY(-4px); background: rgba(255, 255, 255, 0.08); }
.svc-chip.is-active { border-color: var(--color-acento); background: rgba(89, 173, 71, 0.16); box-shadow: 0 10px 30px rgba(89, 173, 71, 0.18); }
.svc-chip__icon { font-size: 2.4rem; }
.svc-chip__title { font-family: var(--fuente-display); text-transform: uppercase; font-size: 1.1rem; letter-spacing: 0.03em; line-height: 1.22; }
.svc-chip__price { font-size: 0.84rem; font-weight: 700; color: var(--color-acento); text-transform: uppercase; letter-spacing: 0.04em; }

/* white detail panel */
.svc-detail {
  display: grid;
  grid-template-columns: 1.05fr 1fr;
  background: #fff;
  border-radius: var(--radio);
  overflow: hidden;
  box-shadow: 0 30px 70px rgba(0, 0, 0, 0.4);
}
.svc-detail__gallery { position: relative; min-height: 380px; background: linear-gradient(135deg, #0e2733, #06141c); }
.svc-detail__photo { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; opacity: 0; transition: opacity 0.9s ease; }
.svc-detail__photo.is-active { opacity: 1; }
.svc-detail__empty { position: absolute; inset: 0; display: flex; align-items: center; justify-content: center; font-size: 5rem; opacity: 0.85; }
.svc-detail__badge {
  position: absolute; top: 18px; left: 18px;
  width: 48px; height: 48px; border-radius: 12px;
  background: var(--color-acento);
  color: #fff;
  display: flex; align-items: center; justify-content: center;
  font-size: 1.4rem; box-shadow: var(--sombra);
}
.svc-detail__dots { position: absolute; bottom: 16px; left: 50%; transform: translateX(-50%); display: flex; gap: 8px; }
.svc-detail__dot { width: 9px; height: 9px; border-radius: 50%; border: none; cursor: pointer; background: rgba(255, 255, 255, 0.45); transition: var(--transicion); padding: 0; }
.svc-detail__dot.is-active { background: #fff; width: 22px; border-radius: 5px; }
.svc-detail__info { padding: 44px; display: flex; flex-direction: column; color: var(--color-texto); }
.svc-detail__price { font-size: 0.8rem; font-weight: 700; color: var(--color-acento-hover); text-transform: uppercase; letter-spacing: 0.05em; margin-bottom: 10px; }
.svc-detail__title { font-family: var(--fuente-display); text-transform: uppercase; font-size: clamp(1.5rem, 3vw, 2.1rem); line-height: 1.05; margin-bottom: 14px; color: var(--color-texto); }
.svc-detail__list li { color: var(--color-texto); }
.svc-detail__desc { color: var(--color-texto-suave); margin-bottom: 22px; }
.svc-detail__list { margin-bottom: 28px; }
.svc-detail__cta { align-self: flex-start; margin-top: auto; }

/* service list checks (shared) */
.service__list { display: grid; gap: 10px; }
.service__list li { position: relative; padding-left: 26px; font-size: 0.95rem; }
.service__list li::before { content: '✓'; position: absolute; left: 0; color: var(--color-acento); font-weight: 700; }

/* ---------- EXPERIENCE BAND ---------- */
.exp__grid { display: grid; grid-template-columns: 1fr 1fr; gap: 64px; align-items: center; }
.exp__title {
  font-family: var(--fuente-display);
  text-transform: uppercase;
  font-size: clamp(1.9rem, 4vw, 3rem);
  font-weight: 700;
  line-height: 1.05;
  margin-bottom: 20px;
}
.exp__body { color: var(--color-dark-texto); margin-bottom: 24px; font-size: 1.05rem; }
.exp__list { display: grid; gap: 12px; margin-bottom: 32px; }
.exp__list li { position: relative; padding-left: 28px; color: #fff; }
.exp__list li::before { content: '✓'; position: absolute; left: 0; color: var(--color-acento); font-weight: 700; }
.exp__img {
  position: relative;
  overflow: hidden;
  padding: 8px;
  border-radius: 18px;
  background: linear-gradient(145deg, rgba(255, 255, 255, 0.06), rgba(255, 255, 255, 0.015));
  border: 1px solid var(--color-dark-borde);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.06), 0 24px 56px rgba(0, 0, 0, 0.42);
  transition: border-color var(--transicion), transform var(--transicion), box-shadow var(--transicion);
}
.exp__img::before {
  content: '';
  position: absolute;
  left: 0; top: 0; bottom: 0;
  width: 4px;
  background: linear-gradient(180deg, var(--color-acento), var(--color-primario));
  transform: scaleY(0);
  transform-origin: top;
  transition: transform var(--transicion);
  z-index: 2;
}
.exp__img:hover {
  transform: translateY(-4px);
  border-color: rgba(89, 173, 71, 0.5);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.06), 0 28px 60px rgba(0, 0, 0, 0.5);
}
.exp__img:hover::before { transform: scaleY(1); }
.exp__img img {
  width: 100%;
  aspect-ratio: 4 / 5;
  object-fit: cover;
  object-position: center;
  border-radius: 12px;
  display: block;
}

/* spacing for the experience band merged below the About cards */
.about-exp {
  position: relative;
  z-index: 1;
  margin-top: 64px;
  padding-top: 56px;
  border-top: 1px solid var(--color-dark-borde);
  align-items: center;
}

/* ---------- ABOUT + TRUST (two columns, elevated) ---------- */
.about-sec { position: relative; overflow: hidden; }
.about-glow {
  position: absolute;
  border-radius: 50%;
  filter: blur(90px);
  z-index: 0;
  pointer-events: none;
}
.about-glow--1 { width: 460px; height: 460px; top: -140px; left: -120px; background: rgba(89, 173, 71, 0.16); }
.about-glow--2 { width: 520px; height: 520px; bottom: -200px; right: -160px; background: rgba(0, 118, 178, 0.16); }
.about2 { position: relative; z-index: 1; display: grid; grid-template-columns: 1fr 1.05fr; gap: 60px; align-items: start; }
.about2__info { position: sticky; top: 100px; }
.about2__title {
  font-family: var(--fuente-display);
  text-transform: uppercase;
  font-size: clamp(1.9rem, 4vw, 3rem);
  font-weight: 700;
  line-height: 1.05;
  color: #fff;
  margin: 12px 0 22px;
  padding-bottom: 18px;
  position: relative;
}
.about2__title::after {
  content: '';
  position: absolute;
  left: 0; bottom: 0;
  width: 72px; height: 4px;
  border-radius: 4px;
  background: linear-gradient(90deg, var(--color-acento), var(--color-primario));
}
.about2__lead {
  font-size: 1.18rem; font-weight: 500; color: #fff;
  margin-bottom: 16px;
  padding-left: 18px;
  border-left: 3px solid var(--color-acento);
}
.about2__body { color: var(--color-dark-texto); font-size: 1.02rem; margin-bottom: 24px; }
.about2__chips { display: flex; flex-wrap: wrap; gap: 10px; margin-bottom: 30px; }
.about2__chips li {
  font-size: 0.82rem; font-weight: 600; color: #d7e6ee;
  padding: 7px 14px 7px 30px;
  border: 1px solid var(--color-dark-borde);
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.04);
  position: relative;
}
.about2__chips li::before {
  content: '✓';
  position: absolute; left: 13px; top: 50%; transform: translateY(-50%);
  color: var(--color-acento); font-weight: 700;
}
.about2__actions { display: flex; gap: 14px; flex-wrap: wrap; }

.about2__cards { display: flex; flex-direction: column; gap: 16px; }
.tcard {
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: flex-start;
  gap: 18px;
  background: linear-gradient(145deg, rgba(255, 255, 255, 0.06), rgba(255, 255, 255, 0.015));
  border: 1px solid var(--color-dark-borde);
  border-radius: 16px;
  padding: 20px 22px;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.06);
  transition: border-color var(--transicion), transform var(--transicion), box-shadow var(--transicion);
}
/* accent stripe that reveals on hover */
.tcard::before {
  content: '';
  position: absolute;
  left: 0; top: 0; bottom: 0;
  width: 4px;
  background: linear-gradient(180deg, var(--color-acento), var(--color-primario));
  transform: scaleY(0);
  transform-origin: top;
  transition: transform var(--transicion);
}
.tcard:hover {
  transform: translateY(-4px);
  border-color: rgba(89, 173, 71, 0.5);
  box-shadow: 0 18px 38px rgba(0, 0, 0, 0.38);
}
.tcard:hover::before { transform: scaleY(1); }
.tcard__num {
  position: absolute;
  top: 14px; right: 18px;
  font-family: var(--fuente-display);
  font-size: 1.1rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.08);
}
.tcard__icon {
  flex-shrink: 0;
  font-size: 1.5rem;
  width: 54px; height: 54px;
  display: flex; align-items: center; justify-content: center;
  border-radius: 14px;
  background: linear-gradient(145deg, rgba(89, 173, 71, 0.32), rgba(0, 118, 178, 0.22));
  border: 1px solid rgba(89, 173, 71, 0.32);
  box-shadow: 0 6px 16px rgba(89, 173, 71, 0.18);
}
.tcard__text h4 { font-size: 1.06rem; color: #fff; margin-bottom: 5px; }
.tcard__text p { color: var(--color-dark-texto); font-size: 0.92rem; line-height: 1.5; }

/* ---------- CTA BANNER ---------- */

/* ---------- FAQ (accordion) ---------- */
.faq__list { max-width: 760px; margin: 0 auto; display: grid; gap: 14px; }
.faq__details {
  background: var(--color-dark-2);
  border: 1px solid var(--color-dark-borde);
  border-radius: 14px;
  overflow: hidden;
  transition: border-color var(--transicion), box-shadow var(--transicion);
}
.faq__details[open] {
  border-color: rgba(89, 173, 71, 0.5);
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.28);
}
.faq__q {
  list-style: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 18px 22px;
  font-weight: 600;
  font-size: 1.04rem;
  color: #fff;
}
.faq__q::-webkit-details-marker { display: none; }
.faq__icon { position: relative; flex-shrink: 0; width: 20px; height: 20px; }
.faq__icon::before,
.faq__icon::after {
  content: '';
  position: absolute;
  top: 50%; left: 50%;
  background: var(--color-acento);
  transform: translate(-50%, -50%);
  border-radius: 2px;
}
.faq__icon::before { width: 14px; height: 2px; }
.faq__icon::after { width: 2px; height: 14px; transition: transform var(--transicion); }
.faq__details[open] .faq__icon::after { transform: translate(-50%, -50%) scaleY(0); }
.faq__a {
  padding: 0 22px 20px;
  color: var(--color-dark-texto);
  font-size: 0.96rem;
  line-height: 1.65;
}

/* ---------- QUOTE FORM (professional) ---------- */
.quote-sec { position: relative; overflow: hidden; }
.quote-glow { position: absolute; border-radius: 50%; filter: blur(100px); z-index: 0; pointer-events: none; }
.quote-glow--1 { width: 460px; height: 460px; top: -160px; left: -120px; background: rgba(89, 173, 71, 0.14); }
.quote-glow--2 { width: 520px; height: 520px; bottom: -200px; right: -160px; background: rgba(0, 118, 178, 0.14); }
.quote { position: relative; z-index: 1; display: grid; grid-template-columns: 1fr 1.15fr; gap: 56px; align-items: center; }
.quote__title { font-family: var(--fuente-display); text-transform: uppercase; font-size: clamp(1.9rem, 4vw, 2.8rem); font-weight: 700; margin: 10px 0 14px; color: #fff; }
.quote__sub { color: var(--color-dark-texto); font-size: 1.08rem; margin-bottom: 26px; }

/* perks */
.quote__perks { display: grid; gap: 12px; margin-bottom: 30px; }
.quote__perks li { display: flex; align-items: center; gap: 12px; color: #fff; font-weight: 500; }
.quote__perk-ico {
  flex-shrink: 0;
  width: 24px; height: 24px;
  display: flex; align-items: center; justify-content: center;
  border-radius: 50%;
  background: rgba(89, 173, 71, 0.18);
  border: 1px solid rgba(89, 173, 71, 0.4);
  color: var(--color-acento);
  font-size: 0.8rem; font-weight: 700;
}

/* contact rows */
.quote__contact { display: grid; gap: 12px; }
.qcontact {
  display: flex; align-items: center; gap: 14px;
  padding: 12px 14px;
  border: 1px solid var(--color-dark-borde);
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.03);
  transition: var(--transicion);
}
a.qcontact:hover { border-color: var(--color-acento); background: rgba(89, 173, 71, 0.08); }
.qcontact__ico {
  flex-shrink: 0;
  width: 42px; height: 42px;
  display: flex; align-items: center; justify-content: center;
  border-radius: 11px;
  font-size: 1.1rem;
  background: linear-gradient(145deg, rgba(89, 173, 71, 0.28), rgba(0, 118, 178, 0.2));
  border: 1px solid rgba(89, 173, 71, 0.3);
}
.qcontact__txt { display: flex; flex-direction: column; line-height: 1.3; }
.qcontact__txt small { font-size: 0.72rem; text-transform: uppercase; letter-spacing: 0.05em; color: var(--color-dark-texto); }
.qcontact__txt strong { font-size: 1rem; font-weight: 600; color: #fff; }

/* form card */
.quote__form {
  position: relative;
  display: flex; flex-direction: column; gap: 16px;
  background: #fff; color: var(--color-texto);
  padding: 34px 32px 30px; border-radius: 18px;
  box-shadow: 0 36px 80px rgba(0, 0, 0, 0.45);
  overflow: hidden;
}
/* top accent bar */
.quote__form::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0;
  height: 5px;
  background: linear-gradient(90deg, var(--color-acento), var(--color-primario));
}
.quote__form-head { margin-bottom: 4px; }
.quote__form-head h3 {
  font-family: var(--fuente-display); text-transform: uppercase;
  font-size: 1.4rem; color: var(--color-texto); margin-bottom: 4px;
}
.quote__form-head p { font-size: 0.9rem; color: var(--color-texto-suave); }
.quote__bolt { color: var(--color-acento-hover); }

.quote__row { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
.quote__form label { display: flex; flex-direction: column; gap: 7px; font-weight: 600; font-size: 0.86rem; color: var(--color-texto); }
.req { color: var(--color-acento-hover); }
.opt { color: var(--color-texto-suave); font-weight: 500; }
.quote__form input, .quote__form select, .quote__form textarea {
  font-family: inherit; font-size: 1rem; padding: 13px 15px;
  border: 1.5px solid var(--color-borde); border-radius: 11px;
  background: var(--color-fondo-alt);
  transition: border-color var(--transicion), box-shadow var(--transicion), background var(--transicion);
  resize: vertical; color: var(--color-texto);
}
.quote__form input::placeholder, .quote__form textarea::placeholder { color: #9fb3bf; }
.quote__form input:focus, .quote__form select:focus, .quote__form textarea:focus {
  outline: none;
  border-color: var(--color-acento);
  background: #fff;
  box-shadow: 0 0 0 3px rgba(89, 173, 71, 0.16);
}
.quote__form select.is-placeholder { color: #9fb3bf; }
.quote__submit {
  margin-top: 6px;
  gap: 10px;
  font-size: 1.02rem;
  padding: 15px 24px;
  box-shadow: 0 12px 28px rgba(89, 173, 71, 0.32);
}
.quote__submit-arrow { transition: transform var(--transicion); }
.quote__submit:hover .quote__submit-arrow { transform: translateX(4px); }
.quote__reassure { font-size: 0.8rem; color: var(--color-texto-suave); text-align: center; margin-top: 2px; }
.quote__ok { color: var(--color-acento-hover); font-weight: 600; text-align: center; }
.quote__err { color: #c0392b; font-weight: 600; text-align: center; }
.quote__submit:disabled { opacity: 0.7; cursor: default; box-shadow: none; }

/* ---------- Light button on dark ---------- */
.btn--light { background: transparent; color: #fff; border-color: rgba(255, 255, 255, 0.4); }
.btn--light:hover { background: rgba(255, 255, 255, 0.12); border-color: #fff; color: #fff; }

@media (max-width: 900px) {
  .exp__grid { grid-template-columns: 1fr; gap: 40px; }
  .quote { grid-template-columns: 1fr; gap: 40px; }
  .about2 { grid-template-columns: 1fr; gap: 36px; }
  .about2__info { position: static; }
  .hero__inner { grid-template-columns: 1fr; gap: 48px; text-align: center; }
  .hero__text { max-width: 100%; margin: 0 auto; }
  .hero__actions, .hero__trust { justify-content: center; }
  .hero__trust { margin-bottom: 0; }
  .hero__media { order: -1; }
  .svc-detail { grid-template-columns: 1fr; }
  .svc-detail__gallery { min-height: 280px; }
  /* no hero overlap when stacked */
  .svc-marquee { margin-top: 28px; }
}
@media (max-width: 600px) {
  .hero { min-height: 78vh; padding: 56px 0; }
  .hero__media { width: min(300px, 75vw); }
  .hero__badge { width: 76px; height: 76px; }
  .quote__row { grid-template-columns: 1fr; }
  .svc-chip { width: 200px; }
  .svc-detail__info { padding: 32px 24px; }
}
</style>
