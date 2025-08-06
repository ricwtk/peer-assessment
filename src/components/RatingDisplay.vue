<template>
  <div class="flex flex-col gap-1">
    <div class="text-xs">{{ props.label }}</div>
    <ProgressBar :value="props.rating" :pt="barcss"></ProgressBar>
  </div>
</template>

<script setup>
const props = defineProps({
  label: String,
  rating: Number
})
import { computed } from 'vue'
const colors = [
  "oklch(63.7% 0.237 25.331)",
  "oklch(70.5% 0.213 47.604)",
  "oklch(76.9% 0.188 70.08)",
  "oklch(79.5% 0.184 86.047)",
  "oklch(76.8% 0.233 130.85)",
  "oklch(72.3% 0.219 149.579)",
  "oklch(69.6% 0.17 162.48)"
]
const barcss = computed(() => {
  let color = colors[Math.round(props.rating/100*colors.length)]
  return { value: { style: `background: ${color}` } }
})
</script>