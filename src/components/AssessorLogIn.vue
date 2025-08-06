<template>
  <div class="flex flex-col gap-2">
    <InputGroup>
      <InputGroupAddon><i class="pi pi-warehouse"></i></InputGroupAddon>
      <Select placeholder="Course" v-model="selectedCourse" :options="courselist" optionLabel="display"></Select>
    </InputGroup>

    <div class="flex flex-row gap-2">
      <InputGroup>
        <InputGroupAddon><i class="pi pi-user"></i></InputGroupAddon>
        <Select placeholder="Yourself" v-model="selectedPerson" :options="personlist" optionLabel="display"></Select>
      </InputGroup>

      <InputGroup>
        <InputGroupAddon><i class="pi pi-key"></i></InputGroupAddon>
        <InputText :useGrouping="false" placeholder="Pin" v-model="providedPin"/>
      </InputGroup>
    </div>

    <Button class="self-end" @click="loadmembers">Load Member List</Button>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const props = defineProps(["courses"])
const courselist = computed(() => props.courses.map(c => {
  c.display = `${c.course_code} ${c.course_name}`
  return c
}))
const personlist = computed(() => {
  if (selectedCourse.value == "") { return [] }
  else {
    return selectedCourse.value.students.map(s => {
      s.display = `${s.id} ${s.name}`
      return s
    })
  }
})

const selectedCourse = ref("")
const selectedPerson = ref("")
const providedPin = ref("")

const emit = defineEmits(["defineassessor"])
const loadmembers = () => {
  emit("defineassessor", {
    course: { 
      key: selectedCourse.value.course_key, 
      name: selectedCourse.value.course_name, 
      code: selectedCourse.value.course_code ,
      lecturers: selectedCourse.value.course_lecturers.map(l => ({ name: l.name, email: l.email }))
    },
    assessor: { 
      id: selectedPerson.value.id,
      name: selectedPerson.value.name
    },
    pin: providedPin.value
  })
}
</script>