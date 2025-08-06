<template>
  <Panel>
    <template #header>
      <div class="grow flex flex-row gap-2">
        <Tag v-for="member in props.assessment.members">
          <div class="flex flex-col">
            <span>{{ member.name }}</span>
            <span class="text-xs">{{ member.id }}</span>
          </div>
        </Tag>
      </div>
      <Button icon="pi pi-times" @click="remove"/>
    </template>
    <div class="grid gap-2 grid-cols-1 sm:grid-cols-2">
      <RatingDisplay label="Contribution" :rating="props.assessment.rating.contribution"/>
      <RatingDisplay label="Teamwork" :rating="props.assessment.rating.teamwork"/>
      <RatingDisplay label="Reliability" :rating="props.assessment.rating.reliability"/>
      <RatingDisplay label="Quality" :rating="props.assessment.rating.quality"/>
    </div>
    {{ props.assessment.justification }}
  </Panel>
</template>

<script setup>
const props = defineProps(["assessment"])

import { computed } from 'vue'
import RatingDisplay from './RatingDisplay.vue'
const colors = [
  "oklch(63.7% 0.237 25.331)",
  "oklch(70.5% 0.213 47.604)",
  "oklch(76.9% 0.188 70.08)",
  "oklch(79.5% 0.184 86.047)",
  "oklch(76.8% 0.233 130.85)",
  "oklch(72.3% 0.219 149.579)",
  "oklch(69.6% 0.17 162.48)"
]
const barcolor = computed(() => colors[Math.round(props.assessment.rating/100*colors.length)])
const getbarcss = (rating) => {
  let color = colors[Math.round(rating/100*colors.length)]
  return { value: { style: `background: ${color}` } }
}
const emit = defineEmits(["remove"])
const remove = () => emit("remove")
</script>