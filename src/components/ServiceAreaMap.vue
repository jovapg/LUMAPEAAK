<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'
import content from '../data/content.json'

const { areas, company } = content
const el = ref(null)
let map

const RED = '#e63946'
const BLUE = '#0076b2'
const GREEN = '#59ad47'

function pinSvg(color, scale = 1) {
  const w = Math.round(28 * scale)
  const h = Math.round(40 * scale)
  return {
    html: `
      <svg width="${w}" height="${h}" viewBox="0 0 24 34" xmlns="http://www.w3.org/2000/svg">
        <path d="M12 0C5.37 0 0 5.37 0 12c0 8.5 12 22 12 22s12-13.5 12-22C24 5.37 18.63 0 12 0z"
              fill="${color}"/>
        <circle cx="12" cy="12" r="4" fill="rgba(0,0,0,0.22)"/>
      </svg>`,
    w, h,
  }
}

onMounted(() => {
  const center = [areas.center.lat, areas.center.lng]

  map = L.map(el.value, {
    center,
    zoom: 15,
    scrollWheelZoom: false,
    zoomControl: true,
    attributionControl: true,
  })

  L.tileLayer(
    'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png',
    { maxZoom: 19, attribution: '&copy; OpenStreetMap &copy; CARTO' }
  ).addTo(map)

  const radiusMeters = (areas.radiusMiles || 25) * 1609.34
  const ring = L.circle(center, {
    radius: radiusMeters,
    color: GREEN,
    weight: 2,
    dashArray: '8 8',
    fillColor: GREEN,
    fillOpacity: 0.1,
  }).addTo(map)

  areas.cities.forEach((c) => {
    const isCenter = c.name === areas.center.name
    const p = pinSvg(isCenter ? BLUE : RED, isCenter ? 0.62 : 0.5)
    const icon = L.divIcon({
      className: 'area-pin',
      html: p.html,
      iconSize: [p.w, p.h],
      iconAnchor: [p.w / 2, p.h], // tip of the pin
    })
    L.marker([c.lat, c.lng], { icon, riseOnHover: true })
      .addTo(map)
      .bindTooltip(`<span class="area-tip__txt">${c.name}</span>`, {
        permanent: true,
        direction: 'right',
        offset: [6, -Math.round(p.h * 0.55)],
        className: 'area-tip',
      })
  })

  map.fitBounds(ring.getBounds(), { padding: [40, 40] })
  map.setZoom(map.getZoom() + 0.5) // start one step closer
})

onBeforeUnmount(() => {
  if (map) map.remove()
})
</script>

<template>
  <div class="area-map-wrap">
    <div ref="el" class="area-map" aria-label="Service area map centered on Seattle"></div>
    <div class="area-map__note">
      <p>{{ company.serviceRadius }}</p>
      <a :href="`tel:${company.phoneLink}`" class="btn btn--primario">📞 Call to Verify</a>
    </div>
  </div>
</template>

<style scoped>
/* isolate so Leaflet's high z-index panes/controls stay contained and never
   paint over the sticky nav */
.area-map-wrap { position: relative; isolation: isolate; }
.area-map {
  height: 560px;
  width: 100%;
  border-top: 1px solid var(--color-dark-borde);
  border-bottom: 1px solid var(--color-dark-borde);
}
.area-map__note {
  position: absolute;
  top: 50%;
  right: 28px;
  transform: translateY(-50%);
  z-index: 1000;
  width: min(340px, calc(100% - 56px));
  text-align: center;
  background: rgba(8, 22, 30, 0.9);
  backdrop-filter: blur(6px);
  border: 1px solid var(--color-dark-borde);
  border-radius: var(--radio);
  padding: 26px 24px;
  box-shadow: 0 18px 44px rgba(0, 0, 0, 0.5);
}
.area-map__note p { color: #fff; margin-bottom: 18px; font-size: 0.95rem; }

@media (max-width: 768px) {
  .area-map__note {
    position: static;
    transform: none;
    width: auto;
    max-width: 560px;
    margin: 20px auto 0;
    background: none;
    backdrop-filter: none;
    border: none;
    box-shadow: none;
    padding: 0 24px;
  }
  .area-map__note p { color: var(--color-dark-texto); }
}
</style>

<style>
/* pin marker: drop the default leaflet box look */
.area-pin { background: none; border: none; }
.area-pin svg { filter: drop-shadow(0 3px 5px rgba(0, 0, 0, 0.5)); display: block; }

/* labels: plain text only — no box, border or shadow */
.area-tip.leaflet-tooltip {
  background: none;
  border: none;
  color: #fff;
  font-family: var(--fuente, system-ui, sans-serif);
  font-weight: 600;
  font-size: 0.9rem;
  letter-spacing: 0.005em;
  padding: 0;
  border-radius: 0;
  box-shadow: none;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.95), 0 0 10px rgba(0, 0, 0, 0.8);
  transition: font-size 0.13s ease, color 0.13s ease;
}
.area-tip.leaflet-tooltip::before { display: none; } /* hide pointer arrow */

/* grow + bring to front on hover */
.area-tip.leaflet-tooltip:hover {
  font-size: 0.95rem;
  color: var(--color-acento);
  z-index: 1000 !important;
}
.area-tip__txt { display: inline-block; white-space: nowrap; }

.leaflet-container { background: #0a1822; font-family: inherit; }
</style>
