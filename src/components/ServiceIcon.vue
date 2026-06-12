<script setup>
defineProps({ name: { type: String, default: 'window' } })
</script>

<template>
  <svg class="svc-ic" viewBox="0 0 24 24" fill="none" stroke="currentColor"
       stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
    <!-- Exterior window -->
    <template v-if="name === 'window'">
      <rect x="3.5" y="3.5" width="17" height="17" rx="1.5" />
      <path d="M12 3.5v17M3.5 12h17" />
    </template>

    <!-- Interior window (sparkle) -->
    <template v-else-if="name === 'spray'">
      <rect x="3" y="4.5" width="13" height="15" rx="1.5" />
      <path d="M9.5 4.5v15M3 12h13" />
      <path d="M19 3.5l.85 2.4 2.4.85-2.4.85L19 10l-.85-2.4L15.75 6.75l2.4-.85z" fill="currentColor" stroke="none" />
    </template>

    <!-- Roof / house -->
    <template v-else-if="name === 'roof'">
      <path d="M2.5 11.5 12 4l9.5 7.5" />
      <path d="M5 10v9.5h14V10" />
      <path d="M10 19.5v-5h4v5" />
    </template>

    <!-- Droplet (soft wash) -->
    <template v-else-if="name === 'droplet'">
      <path d="M12 3.2s6.2 6.6 6.2 10.6a6.2 6.2 0 1 1-12.4 0C5.8 9.8 12 3.2 12 3.2z" />
    </template>

    <!-- Leaf (gutter) -->
    <template v-else-if="name === 'leaf'">
      <path d="M4 20c0-9 6.5-15 16-15 0 9.5-6 16-16 15z" />
      <path d="M9 15c2.8-3 5-4.2 8.5-5.2" />
    </template>

    <!-- Solar panel -->
    <template v-else-if="name === 'solar'">
      <rect x="3" y="4.5" width="18" height="11" rx="1" />
      <path d="M3 8.2h18M3 11.9h18M9 4.5v11M15 4.5v11" />
      <path d="M12 15.5v3M8.5 21h7" />
    </template>

    <!-- Sun (skylight / light) -->
    <template v-else-if="name === 'sun'">
      <circle cx="12" cy="12" r="4" />
      <path d="M12 2v2.5M12 19.5V22M4.2 4.2l1.8 1.8M18 18l1.8 1.8M2 12h2.5M19.5 12H22M4.2 19.8 6 18M18 6l1.8-1.8" />
    </template>

    <!-- Holiday lights -->
    <template v-else-if="name === 'lights'">
      <path d="M2.5 5c3.2 3.2 6.3 3.2 9.5 0s6.3-3.2 9.5 0" />
      <path d="M6 6.6l-.6 3M12 6.6l-.6 3M18 6.6l-.6 3" />
      <circle cx="5.4" cy="11.4" r="1.9" fill="currentColor" stroke="none" />
      <circle cx="11.4" cy="11.4" r="1.9" fill="currentColor" stroke="none" />
      <circle cx="17.4" cy="11.4" r="1.9" fill="currentColor" stroke="none" />
    </template>
  </svg>
</template>

<style scoped>
.svc-ic { width: 1em; height: 1em; display: block; }
</style>
