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
      <div class="flex flex-col gap-2">
        <AssessorLogIn :courses="courses" @defineassessor="loadmembers"/>
        <Message severity="error" v-if="loginerror">{{ loginerror }}</Message>
      </div>
    </Panel>
    <Panel v-if="assessor.id !== ''">
      <template #header>
        <div class="flex flex-row gap-2 items-center">
          <Tag>Step 2</Tag>
          <div class="font-bold">Peer Assessment</div>
        </div>
      </template>
      <Message severity="success">I am {{ assessor.name }} ({{ assessor.id }}) reviewing members in {{ assessingcourse.code }} {{ assessingcourse.name }}</Message>
      <AssessForm :pendinglist="pendinglist" @save="addassessment"/>
    </Panel>
    <Panel pt:content:class="flex flex-col gap-2" v-if="assessor.id !== ''">
      <template #header>
        <div class="flex flex-row gap-2 items-center">
          <Tag>Step 3</Tag>
          <div class="font-bold">Submit Assessment</div>
        </div>
      </template>
      <AssessmentDisplay v-for="asm in assessmentlist" :assessment="asm"></AssessmentDisplay>
      <Button class="self-end" @click="">Submit Assessment</Button>
    </Panel>
  </div>
</template>

<script setup>
import { API_BASE_URL } from './config';
import AssessForm from './components/AssessForm.vue';
import AssessmentDisplay from './components/AssessmentDisplay.vue';
import AssessorLogIn from './components/AssessorLogIn.vue';
import TopBar from './components/TopBar.vue';
import { ref, computed } from "vue"

const courses = ref([])
async function fetchCourses() {
  try {
    const response = await fetch(`${API_BASE_URL}/api/courses`);
    const data = await response.json();
    console.log("Courses:", data);
    courses.value = data
  } catch (error) {
    console.error("Error fetching courses:", error);
  }
}

fetchCourses();

const loginerror = ref("")
async function getStudentGroup(courseKey, studentId, pin) {
  loginerror.value = ""
  try {
    const response = await fetch(`${API_BASE_URL}/api/group`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        course: courseKey,     // e.g., "course1"
        student_id: studentId, // e.g., "S101"
        pin: pin               // e.g., "1234"
      })
    });

    if (!response.ok) {
      const errorData = await response.json();
      loginerror.value = errorData.error || "Unknown error";
      throw new Error(errorData.error || "Unknown error");
    }

    const data = await response.json();
    console.log("Group members:", data.group);
    memberlist.value = data.group.filter(s => s.id !== assessor.value.id)
  } catch (error) {
    loginerror.value = error.message;
    console.error("Error fetching group:", error.message);
  }
}

const loadmembers = (login) => {
  assessor.value.id = login.assessor.id
  assessor.value.name = login.assessor.name
  assessingcourse.value.key = login.course.key
  assessingcourse.value.code = login.course.code
  assessingcourse.value.name = login.course.name
  getStudentGroup(login.course.key, login.assessor.id, login.pin);
}

const assessor = ref({
  id: "", name: ""
})

const assessingcourse = ref({
  key: "",
  code: "",
  name: ""
})

const memberlist = ref([])

const pendinglist = computed(() => memberlist.value.filter(m => {
  return !assessmentlist.value.map(a => a.members.map(x => x.id).includes(m.id)).some(x => x)
}))

const assessmentlist = ref([])

const addassessment = (asm) => {
  assessmentlist.value.push(asm)
}

</script>