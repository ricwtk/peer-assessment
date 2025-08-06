<template>
  <div class="flex flex-col gap-2">
    <div class="flex flex-row gap-2">
      <InputGroup>
        <InputGroupAddon><i class="pi pi-warehouse"></i></InputGroupAddon>
        <Select placeholder="Course" v-model="selectedCourse" :options="courselist" optionLabel="display"></Select>
      </InputGroup>

      <InputGroup>
        <InputGroupAddon><i class="pi pi-key"></i></InputGroupAddon>
        <InputText :useGrouping="false" placeholder="Admin Pin" v-model="providedPin"/>
      </InputGroup>
    </div>

    <Button class="self-end" @click="loadstudents">Load Student List</Button>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const props = defineProps(["courses"])
const courselist = computed(() => props.courses.map(c => {
  c.display = `${c.course_code} ${c.course_name}`
  return c
}))

const selectedCourse = ref("")
const providedPin = ref("")

const emit = defineEmits(["adminlogin"])
const loadstudents = () => {
  emit("adminlogin", {
    course: { 
      key: selectedCourse.value.course_key, 
      name: selectedCourse.value.course_name, 
      code: selectedCourse.value.course_code ,
    },
    pin: providedPin.value
  })
}
</script>