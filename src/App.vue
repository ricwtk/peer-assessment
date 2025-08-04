<template>
  <div class="w-full max-w-5xl flex flex-col gap-2 m-2">
    <TopBar />
    <Panel>
      <template #header>
        <div class="flex flex-row gap-2 items-center">
          <Tag>Step 1</Tag>
          <div class="font-bold">Your Information</div>
        </div>
      </template>
      <AssessorLogIn />
    </Panel>
    <Panel>
      <template #header>
        <div class="flex flex-row gap-2 items-center">
          <Tag>Step 2</Tag>
          <div class="font-bold">Peer Assessment</div>
        </div>
      </template>
      <AssessForm :pendinglist="pendinglist" @save="addassessment"/>
    </Panel>
    <Panel pt:content:class="flex flex-col gap-2">
      <template #header>
        <div class="flex flex-row gap-2 items-center">
          <Tag>Step 3</Tag>
          <div class="font-bold">Submit Assessment</div>
        </div>
      </template>
      <AssessmentDisplay v-for="asm in assessmentlist" :assessment="asm"></AssessmentDisplay>
    </Panel>
  </div>
</template>

<script setup>
import AssessForm from './components/AssessForm.vue';
import AssessmentDisplay from './components/AssessmentDisplay.vue';
import AssessorLogIn from './components/AssessorLogIn.vue';
import TopBar from './components/TopBar.vue';
import { ref, computed } from "vue"

const memberlist = ref([
  { id: "324134", name: "1234142" },
  { id: "dsfoijhfkij", name: "dsfjalkfj" },
  { id: "d9s87f9ads", name: "fd0978f" },
  { id: "84239", name: "fdafas" },
])

const pendinglist = computed(() => memberlist.value.filter(m => {
  return !assessmentlist.value.map(a => a.members.map(x => x.id).includes(m.id)).some(x => x)
}))

const assessmentlist = ref([])

const addassessment = (asm) => {
  assessmentlist.value.push(asm)
}

</script>