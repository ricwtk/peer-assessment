<template>
  <div class="w-full max-w-5xl flex flex-col gap-2 m-2">
    <TopBar />
    <Stepper value="0" class="basis-[0rem]">
      <StepList>
        <Step value="1">Your Information</Step>
        <Step value="2">Peer Assessment</Step>
        <Step value="3">Submit Assessment</Step>
      </StepList>
    </Stepper>
    <Panel>
      <template #header>
        <div class="flex flex-row gap-2 items-center">
          <Tag>Step 1</Tag>
          <div class="font-bold">Your Information</div>
        </div>
      </template>
      <div class="flex flex-col gap-2">
        <AssessorLogIn :courses="courses" @defineassessor="loadmembers"/>
        <Message severity="error" v-if="loginerror"><strong>Error</strong> {{ loginerror }}</Message>
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
      <AssessmentDisplay v-for="(asm, index) in assessmentlist" :assessment="asm" @remove="deleteassessment(index)"></AssessmentDisplay>
      <Button class="self-end" @click="submitAssessments" :disabled="assessmentlist.length < 1">Submit Assessment</Button>
      <Message severity="success" v-if="submissionsuccess">{{ submissionsuccess }}</Message>
      <Message severity="error" v-if="submissionerror"><strong>Error</strong> {{ submissionerror }}</Message>
    </Panel>
  </div>
</template>

<script setup>
import { API_BASE_URL, SUBMISSION_ENDPOINT } from './config';
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
    courses.value.sort((a, b) => a.course_code.localeCompare(b.course_code))
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
  assessingcourse.value.lecturers = login.course.lecturers.map(l => ({ name: l.name, email: l.email}))
  assessmentlist.value = []
  getStudentGroup(login.course.key, login.assessor.id, login.pin);
}

const assessor = ref({
  id: "", name: ""
})

const assessingcourse = ref({
  key: "",
  code: "",
  name: "",
  lecturers: []
})

const memberlist = ref([])

const pendinglist = computed(() => memberlist.value.filter(m => {
  return !assessmentlist.value.map(a => a.members.map(x => x.id).includes(m.id)).some(x => x)
}))

const assessmentlist = ref([])

const addassessment = (asm) => {
  assessmentlist.value.push(asm)
}
const deleteassessment = (index) => {
  assessmentlist.value.splice(index, 1)
}

const submissionerror = ref("")
const submissionsuccess = ref("")
async function submitAssessments() {
  submissionerror.value = ""
  submissionsuccess.value = ""
  try {
    let req_body = JSON.stringify({
      assessor: { id: assessor.value.id, name: assessor.value.name },
      course: { 
        key: assessingcourse.value.key, 
        code: assessingcourse.value.code, 
        name: assessingcourse.value.name,
        lecturers: assessingcourse.value.lecturers.map(l => ({ name: l.name, email: l.email}))
      }, 
      assessments: assessmentlist.value.map(asm => ({
        members: asm.members.map(x => ({ id: x.id, name: x.name })),
        rating: asm.rating,
        justification: asm.justification
      }))
    })
    // console.log(req_body)
    // console.log(SUBMISSION_ENDPOINT)
    const response = await fetch(`${SUBMISSION_ENDPOINT}`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: req_body
    });

    if (!response.ok) {
      const errorData = await response.json();
      submissionerror.value = errorData.error || "Unknown error";
      throw new Error(errorData.error || "Unknown error");
    } else {
      submissionsuccess.value = "Submission succeeded, you should receive an email with the submitted details."
    }
  } catch (error) {
    submissionerror.value = error.message;
    console.error("Error submitting assessments:", error.message);
  }
}

</script>