<template>
  <div class="w-full max-w-5xl flex flex-col gap-2 m-2">
    <TopBar />
    <Panel>
      <template #header>
        <div class="flex flex-row gap-2 items-center">
          <Tag>Step 1</Tag>
          <div class="font-bold">Provide admin pin for course</div>
        </div>
      </template>
      <div class="flex flex-col gap-2">
        <AdminLogin :courses="courses" @adminlogin="adminlogin"/>
        <Message severity="error" v-if="loginerror"><strong>Error</strong> {{ loginerror }}</Message>
      </div>
    </Panel>
    <Panel v-if="studentlist.length > 0" pt:content:class="flex flex-col gap-2">
      <template #header>
        <div class="flex flex-row gap-2 items-center">
          <Tag>Step 2</Tag>
          <div class="font-bold">Email Template</div>
        </div>
      </template>
      <div class="flex flex-col gap-2">
        <div>Cc <span class="text-xs">(separate emails with semicolon ;)</span></div>
        <InputText class="grow" v-model="cclist"/>
      </div>
      <div class="flex flex-col gap-2">
        <div>Bcc <span class="text-xs">(separate emails with semicolon ;)</span></div>
        <InputText class="grow" v-model="bcclist"/>
      </div>
      <Message severity="info">
        <span>The following template symbols will be substituted by their respective values</span>
        <table><tbody>
          <tr><td class="px-2"><span class="font-mono">%course_code%</span></td><td class="px-2">Course code</td></tr>
          <tr><td class="px-2"><span class="font-mono">%course_name%</span></td><td class="px-2">Course name</td></tr>
          <tr><td class="px-2"><span class="font-mono">%student_id%</span></td><td class="px-2">Student ID</td></tr>
          <tr><td class="px-2"><span class="font-mono">%student_name%</span></td><td class="px-2">Student name</td></tr>
          <tr><td class="px-2"><span class="font-mono">%pin%</span></td><td class="px-2">PIN</td></tr>
        </tbody></table>
      </Message>
      <Editor v-model="emailtemplate" editorStyle="height: 320px" />
      <Editor v-model="sampleemail" editorStyle="height: 320px" readonly pt:toolbar:class="!bg-primary !text-white">
        <template v-slot:toolbar><div class="">Sample email</div></template>
      </Editor>
      <div class="hidden">{{ emailtemplate }}</div>
    </Panel>
    <Panel v-if="studentlist.length > 0" pt:content:class="flex flex-col gap-2">
      <template #header>
        <div class="flex flex-row gap-2 items-center">
          <Tag>Step 3</Tag>
          <div class="font-bold">Email pin to selected students</div>
        </div>
      </template>
      <Message severity="info" v-if="selectedStudents.length == 0">No student is selected</Message>
      <Message severity="info" v-else>{{ selectedStudents.length }} student{{ selectedStudents.length > 1 ? 's are' : ' is' }} selected</Message>
      <DataTable :value="studentlist" v-model:selection="selectedStudents" size="small" stripedRows>
        <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>
        <Column field="id" header="Student ID"></Column>
        <Column field="name" header="Student Name"></Column>
        <Column field="pin" header="PIN"></Column>
      </DataTable>
      <Button class="self-end" @click="sendEmail">Send Email</Button>
      <Message severity="success" v-if="emailsuccess">{{ emailsuccess }}</Message>
      <Message severity="error" v-if="emailerror"><strong>Error</strong> {{ emailerror }}</Message>
    </Panel>
  </div>
</template>

<script setup>
import TopBar from './components/TopBar.vue';
import AdminLogin from './components/AdminLogin.vue';
import { API_BASE_URL, EMAIL_ENDPOINT } from './config';
import { ref, computed } from "vue"

const courses = ref([])
async function fetchCourses() {
  try {
    const response = await fetch(`${API_BASE_URL}/api/coursesonly`);
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
const studentlist = ref([])
async function getStudentList(courseKey, pin) {
  loginerror.value = ""
  try {
    const response = await fetch(`${API_BASE_URL}/api/studentwithpin`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        course: courseKey,     // e.g., "course1"
        pin: pin               // e.g., "1234"
      })
    });

    if (!response.ok) {
      const errorData = await response.json();
      loginerror.value = errorData.error || "Unknown error";
      throw new Error(errorData.error || "Unknown error");
    }

    const data = await response.json();
    selectedCourse.value = data.course
    studentlist.value = Object.keys(data.students).sort().map(k => {
      let x = data.students[k]
      x.id = k
      return x
    })
    console.log("Course", selectedCourse.value)
    console.log("Students:", studentlist.value);
  } catch (error) {
    loginerror.value = error.message;
    console.error("Error fetching group:", error.message);
  }
}

const adminlogin = (login) => {
  getStudentList(login.course.key, login.pin)
}

const selectedCourse = ref({})

const cclist = ref("")
const bcclist = ref("")
const emailtemplate = ref('<p>Hi&nbsp;%student_name%,</p><p></p><p>You&nbsp;are&nbsp;required&nbsp;to&nbsp;complete&nbsp;the&nbsp;peer&nbsp;assessment&nbsp;for&nbsp;%course_code%&nbsp;%course_name%.</p><p></p><ol><li>Go&nbsp;to&nbsp;<a href="https://ricwtk.github.io/peer-assessment/" rel="noopener noreferrer" target="_blank">https://ricwtk.github.io/peer-assessment/</a></li><li>Select&nbsp;%course_code%&nbsp;%course_name%&nbsp;from&nbsp;the&nbsp;&quot;Course&quot;&nbsp;dropdown&nbsp;list</li><li>Select&nbsp;your&nbsp;student&nbsp;ID&nbsp;and&nbsp;name&nbsp;from&nbsp;the&nbsp;&quot;Yourself&quot;&nbsp;dropdown&nbsp;list</li><li>Provide&nbsp;this&nbsp;PIN:&nbsp;<strong><u>%pin%</u></strong></li><li>Add&nbsp;assessments&nbsp;for&nbsp;your&nbsp;members.&nbsp;You&nbsp;may&nbsp;group&nbsp;different&nbsp;members&nbsp;under&nbsp;the&nbsp;same&nbsp;rating&nbsp;and&nbsp;justification</li><li>Submit&nbsp;the&nbsp;peer&nbsp;assessment</li></ol><p></p><p>The&nbsp;average&nbsp;rating&nbsp;for&nbsp;each&nbsp;student&nbsp;will&nbsp;be&nbsp;used&nbsp;to&nbsp;modify&nbsp;the&nbsp;final&nbsp;marks&nbsp;he/she&nbsp;will&nbsp;get.&nbsp;If&nbsp;the&nbsp;group&nbsp;obtains&nbsp;70&nbsp;marks&nbsp;for&nbsp;the&nbsp;assignment,&nbsp;and&nbsp;the&nbsp;student&nbsp;gets&nbsp;90%&nbsp;from&nbsp;the&nbsp;rating,&nbsp;the&nbsp;final&nbsp;marks&nbsp;the&nbsp;student&nbsp;gets&nbsp;will&nbsp;be&nbsp;90%&nbsp;x&nbsp;70&nbsp;marks&nbsp;=&nbsp;63&nbsp;marks.</p><p></p><p>This&nbsp;form&nbsp;must&nbsp;be&nbsp;completed&nbsp;privately&nbsp;and&nbsp;your&nbsp;scores&nbsp;will&nbsp;only&nbsp;be&nbsp;known&nbsp;to&nbsp;the&nbsp;supervisors.&nbsp;Your&nbsp;supervisors&nbsp;may&nbsp;moderate&nbsp;the&nbsp;marks&nbsp;accordingly&nbsp;to&nbsp;ensure&nbsp;its&nbsp;truly&nbsp;reflective&nbsp;of&nbsp;the&nbsp;quality&nbsp;of&nbsp;work&nbsp;submitted.&nbsp;</p><p></p><p>Regards,</p><p>Richard</p>')

const sampleemail = computed(() => 
  emailtemplate.value
  .replaceAll("%course_code%", selectedCourse.value.code)
  .replaceAll("%course_name%", selectedCourse.value.name)
  .replaceAll("%student_id%", "123456")
  .replaceAll("%student_name%", "Tim Stevenson")
  .replaceAll("%pin%", "wx12d4")
)
const selectedStudents = ref([])

const emailerror = ref("")
const emailsuccess = ref("")
async function sendEmail() {
  emailerror.value = ""
  emailsuccess.value = ""
  try {
    let req_body = JSON.stringify({
      course: { 
        key: selectedCourse.value.key, 
        code: selectedCourse.value.code, 
        name: selectedCourse.value.name,
      }, 
      students: selectedStudents.value,
      emailtemplate: emailtemplate.value,
      cclist: cclist.value,
      bcclist: bcclist.value
    })
    // console.log(req_body)
    // console.log(EMAIL_ENDPOINT)
    const response = await fetch(`${EMAIL_ENDPOINT}`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: req_body
    });

    if (!response.ok) {
      const errorData = await response.json();
      emailerror.value = errorData.error || "Unknown error";
      throw new Error(errorData.error || "Unknown error");
    } else {
      emailsuccess.value = "Submission succeeded, the students should receive their pin through their email."
    }
  } catch (error) {
    emailerror.value = error.message;
    console.error("Error submitting email requests:", error.message);
  }
}

</script>