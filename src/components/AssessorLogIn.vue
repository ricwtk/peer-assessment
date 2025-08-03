<template>
  <div class="flex flex-col gap-2">
    <InputGroup>
      <InputGroupAddon><i class="pi pi-warehouse"></i></InputGroupAddon>
      <Select placeholder="Course" v-model="selectedCourse"></Select>
    </InputGroup>

    <div class="flex flex-row gap-2">
      <InputGroup>
        <InputGroupAddon><i class="pi pi-user"></i></InputGroupAddon>
        <Select placeholder="Yourself" v-model="selectedPerson"></Select>
      </InputGroup>

      <InputGroup>
        <InputGroupAddon><i class="pi pi-key"></i></InputGroupAddon>
        <InputNumber :useGrouping="false" placeholder="Pin" v-model="providedPin"/>
      </InputGroup>
    </div>

    <Button class="self-end">Load Member List</Button>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const selectedCourse = ref("")
const selectedPerson = ref("")
const providedPin = ref("")
</script>