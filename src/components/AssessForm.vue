<template>
  <Fieldset legend="Form" pt:content:class="flex flex-col gap-2">
    <div>Pending for rating</div>
    <div class="flex flex-row flex-wrap items-center gap-2">
      <Tag v-if="pendingMembers.length == 0" class="!bg-gray-300 !text-gray-500">
        <div class="flex flex-col">
          <span>All member added</span>
          <span>for rating</span>
        </div>
      </Tag>
      <Tag v-for="member in pendingMembers">
        <div class="flex flex-col">
          <span>{{ member.name }}</span>
          <span class="text-xs">{{ member.id }}</span>
        </div>
        <Button icon="pi pi-plus" @click="addforrating(member)"/>
      </Tag>
    </div>
    <div>Rating for</div>
    <div class="flex flex-row flex-wrap items-center gap-2">
      <Tag v-if="selectedMembers.length == 0" class="!bg-gray-300 !text-gray-500">
        <div class="flex flex-col">
          <span>Add member from</span>
          <span>the list above</span>
        </div>
      </Tag>
      <Tag v-for="(member, index) in selectedMembers">
        <div class="flex flex-col">
          <span>{{ member.name }}</span>
          <span class="text-xs">{{ member.id }}</span>
        </div>
        <Button icon="pi pi-times" @click="removefromrating(index)"/>
      </Tag>
    </div>
    <InputGroup class="justify-stretch">
      <Button v-for="x in 21" class="!text-xs grow shrink !border-0" 
        @click="setrating(x)" 
        :class="getclassforratingbutton(x)">
        {{ convertcounttopercentage(x) }}
      </Button>
    </InputGroup>
    <InputGroup class="justify-stretch">
      <Button disabled v-for="x in ratingColorN+1" class="!text-xs grow shrink !border-0"
        :class="'rating-color-'+(x-1)">
      </Button>
    </InputGroup>
    <Textarea rows="5" cols="30" placeholder="Justification" v-model="justificationText"/>
    <Button class="self-end">Add Assessment</Button>
  </Fieldset>
</template>

<script setup>
import { computed, ref } from 'vue';
const props = defineProps({
  pendinglist: Array,
})

const selectedMembers = ref([])
const addforrating = (m) => {
  selectedMembers.value.push(m)
}
const pendingMembers = computed(() => props.pendinglist.filter((m) => !selectedMembers.value.map(x => x.id).includes(m.id)))
const removefromrating = (i) => {
  selectedMembers.value.splice(i,1)
}
const selectedRating = ref(0)
const convertcounttopercentage = (count) => (count-1)*5
const setrating = (count) => {
  selectedRating.value = convertcounttopercentage(count)
}
const ratingColorN = 6
const getclassforratingbutton = (count) => {
  let rating = convertcounttopercentage(count)
  if (rating > selectedRating.value) { return "!bg-white !text-primary" }
  else {
    let cls = "!text-white"
    let indexforcolor = Math.round(selectedRating.value/100*ratingColorN)
    return `${cls} rating-color-${indexforcolor}`
  }
}
const justificationText = ref("")
</script>

<style scoped>
@reference "tailwindcss";
.rating-color-0 { @apply bg-red-500 }
.rating-color-1 { @apply bg-orange-500 }
.rating-color-2 { @apply bg-amber-500 }
.rating-color-3 { @apply bg-yellow-500 }
.rating-color-4 { @apply bg-lime-500 }
.rating-color-5 { @apply bg-green-500 }
.rating-color-6 { @apply bg-emerald-500 }
</style>