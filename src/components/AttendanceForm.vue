<script setup>
import { ref, onMounted, watch, computed } from 'vue'

// Form data refs
const date = ref('')
const day = ref('')
const time = ref('')
const subject = ref('')
const subjects = ref([])
const professor = ref('')
const building = ref('')
const classroom = ref('')
const name = ref('')
const email = ref('')
const studentnumber = ref('')
const yearsection = ref('')
const remarks = ref('attendance')
const terminalno = ref('0')

// Form state
const isSubmitting = ref(false)
const submitError = ref('')
const submitSuccess = ref(false)

// Validate student number format
const validateStudentNumber = (studentNum) => {
  // Regular expression to match the format: two digits, dash, four digits
  const studentNumberRegex = /^\d{2}-\d{4}$/
  return studentNumberRegex.test(studentNum)
}

// Email validation function
const validateEmail = (email) => {
  // Comprehensive email validation regex
  const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/
  return emailRegex.test(email)
}

// Update date and time
const updateDateTime = () => {
  const now = new Date()
  date.value = now.toISOString().split('T')[0]
  day.value = now.toLocaleDateString('en-US', { weekday: 'long' })
  time.value = now.toLocaleTimeString('en-US', {
    hour12: false,
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
}

// Fetch subjects from the backend
const fetchSubjects = async () => {
  try {
    const response = await fetch('http://localhost:8000/api/subjects')
    if (!response.ok) throw new Error('Failed to fetch subjects')
    subjects.value = await response.json()
  } catch (error) {
    console.error('Error fetching subjects:', error)
    submitError.value = 'Failed to fetch subjects'
  }
}

// Watch for changes in subject to populate details
watch(subject, async (newSubject) => {
  if (newSubject) {
    try {
      // Find the selected subject in the subjects array
      const selectedSubjectData = subjects.value.find(s => s.id === newSubject)
      
      if (selectedSubjectData) {
        // Populate section details
        yearsection.value = selectedSubjectData.section?.name || ''
        
        // Populate professor details
        professor.value = selectedSubjectData.professor?.name || ''
        
        // Populate classroom and building details
        building.value = selectedSubjectData.section?.classroom?.building?.name || ''
        classroom.value = selectedSubjectData.section?.classroom?.name || ''
      } else {
        // Reset fields if no matching subject found
        professor.value = ''
        building.value = ''
        classroom.value = ''
        yearsection.value = ''
      }
    } catch (error) {
      console.error('Error fetching subject details:', error)
      submitError.value = 'Failed to fetch subject details'
    }
  }
})

onMounted(() => {
  fetchSubjects()
  updateDateTime()
  setInterval(updateDateTime, 1000)
})

const handleSubmit = async () => {
  try {
    isSubmitting.value = true
    submitError.value = ''
    submitSuccess.value = false

    // Validate student number format
    if (!validateStudentNumber(studentnumber.value)) {
      throw new Error('Student number must be in the format XX-XXXX (e.g., 21-1558)')
    }

    // Validate email if provided
    if (email.value && !validateEmail(email.value)) {
      throw new Error('Please enter a valid email address')
    }

    const formData = {
      subject_id: subject.value,
      terminal_number: terminalno.value,
      student_full_name: name.value,
      student_number: studentnumber.value,
      student_email: email.value || null,
      remarks: remarks.value
    }

    const response = await fetch('http://localhost:8000/api/store/attendance', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify(formData)
    })

    if (!response.ok) {
      const errorData = await response.text()
      throw new Error(errorData || `Server error: ${response.status}`)
    }

    submitSuccess.value = true
    resetForm()
  } catch (error) {
    console.error('Error:', error)
    submitError.value = error.message || 'Failed to record attendance'
  } finally {
    isSubmitting.value = false
  }
}

const resetForm = () => {
  subject.value = ''
  professor.value = ''
  building.value = ''
  classroom.value = ''
  name.value = ''
  email.value = ''
  studentnumber.value = ''
  yearsection.value = ''
  remarks.value = 'attendance'
  terminalno.value = '0'
}
</script>

<template>
  <div class="min-h-screen bg-gray-50 p-4 md:p-6 lg:p-8">
    <div class="mx-auto max-w-4xl bg-white rounded-lg shadow-md p-4 md:p-6">
      <h1 class="text-2xl md:text-3xl font-bold text-gray-800 mb-6 text-center">
        QCU Utilization Monitoring Form
      </h1>

      <!-- Alert Messages -->
      <div v-if="submitSuccess" class="mb-4 p-4 rounded-lg bg-green-50 text-green-700 flex items-center gap-2">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
          <path fill-rule="evenodd"
            d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"
            clip-rule="evenodd" />
        </svg>
        <span>Attendance recorded successfully!</span>
      </div>

      <div v-if="submitError" class="mb-4 p-4 rounded-lg bg-red-50 text-red-700 flex items-center gap-2">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
          <path fill-rule="evenodd"
            d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7 4a1 1 0 11-2 0 1 1 0 012 0zm-1-9a1 1 0 00-1 1v4a1 1 0 102 0V6a1 1 0 00-1-1z"
            clip-rule="evenodd" />
        </svg>
        <span>{{ submitError }}</span>
      </div>

      <form @submit.prevent="handleSubmit" class="space-y-6">
        <!-- Realtime Date and Time -->
        <div class="space-y-2">
          <label class="block text-sm font-medium text-gray-700">Current Date & Time</label>
          <div class="px-3 py-2 border border-gray-300 rounded-lg bg-gray-50 text-gray-800">
            {{ day }}, {{ date }} - {{ time }}
          </div>
        </div>

        <!-- Subject Selection -->
        <div class="space-y-2">
          <label class="block text-sm font-medium text-gray-700">Subject</label>
          <select v-model="subject" required
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
            <option value="" disabled>Select Subject</option>
            <option v-for="subj in subjects" :key="subj.id" :value="subj.id">
              {{ subj.subject_code }} - {{ subj.name }}
            </option>
          </select>
        </div>

        <!-- Readonly Fields for Context -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div class="space-y-2">
            <label class="block text-sm font-medium text-gray-700">Year/Section</label>
            <input type="text" v-model="yearsection" readonly
              class="w-full px-3 py-2 border border-gray-300 rounded-lg bg-gray-50">
          </div>
          <div class="space-y-2">
            <label class="block text-sm font-medium text-gray-700">Professor</label>
            <input type="text" v-model="professor" readonly
              class="w-full px-3 py-2 border border-gray-300 rounded-lg bg-gray-50">
          </div>
        </div>

        <!-- Location Information -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div class="space-y-2">
            <label class="block text-sm font-medium text-gray-700">Building</label>
            <input type="text" v-model="building" readonly
              class="w-full px-3 py-2 border border-gray-300 rounded-lg bg-gray-50">
          </div>
          <div class="space-y-2">
            <label class="block text-sm font-medium text-gray-700">Classroom</label>
            <input type="text" v-model="classroom" readonly
              class="w-full px-3 py-2 border border-gray-300 rounded-lg bg-gray-50">
          </div>
        </div>

        <!-- Student Information -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div class="space-y-2 md:col-span-1">
            <label class="block text-sm font-medium text-gray-700">Full Name</label>
            <input type="text" v-model="name" required
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
          </div>

          <div class="space-y-2 md:col-span-1">
            <label class="block text-sm font-medium text-gray-700">Email (Optional)</label>
            <input 
              type="email" 
              v-model="email"
              pattern="[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"
              placeholder="username@gmail.com"
              title="Please enter a valid email address"
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
            >
            <p class="text-xs text-gray-500 mt-1">Format: username@gmail.com</p>
          </div>

          <div class="space-y-2 md:col-span-1">
            <label class="block text-sm font-medium text-gray-700">Student No</label>
            <input 
              type="text" 
              v-model="studentnumber" 
              required
              pattern="\d{2}-\d{4}"
              placeholder="21-0000"
              title="Student number must be in the format XX-XXXX"
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
            >
            <p class="text-xs text-gray-500 mt-1">Format: XX-XXXX (e.g., 21-0000)</p>
          </div>
        </div>

        <!-- Hidden fields -->
        <input type="hidden" v-model="remarks">
        <input type="hidden" v-model="terminalno">

        <!-- Submit Button -->
        <div class="flex justify-center pt-4">
          <button type="submit" :disabled="isSubmitting || !subject"
            class="px-6 py-2 bg-blue-600 text-white rounded-lg shadow hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed transition-colors">
            {{ isSubmitting ? 'Submitting...' : 'Submit' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>