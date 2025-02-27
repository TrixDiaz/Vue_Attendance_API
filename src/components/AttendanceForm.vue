<script setup>
import { ref, onMounted, watch, computed } from 'vue'

// Form data refs
const date = ref('')
const day = ref('')
const time = ref('')
const subject = ref('')
const professorname = ref('')
const building = ref('')
const classroom = ref('')
const terminalno = ref('')
const name = ref('')
const email = ref('')
const studentnumber = ref('')
const yearsection = ref('')
const components = ref([])
const remarks = ref('')
const sections = ref([])
const selectedSection = ref('')
const isUsingComputer = ref(false) // New ref to track if user is using computer

// Define available components
const availableComponents = ['system unit', 'monitor', 'keyboard', 'mouse', 'network']

// Computed property for the "Select All" checkbox state
const allComponentsSelected = computed({
  get: () => {
    return availableComponents.length > 0 && components.value.length === availableComponents.length
  },
  set: (value) => {
    components.value = value ? [...availableComponents] : []
  }
})

// Form state
const isSubmitting = ref(false)
const submitError = ref('')
const submitSuccess = ref(false)

// Update date and time
const updateDateTime = () => {
  const now = new Date()
  date.value = now.toISOString().split('T')[0]
  day.value = now.toLocaleDateString('en-US', { weekday: 'long' })
  time.value = now.toLocaleTimeString('en-US', {
    hour12: false,
    hour: '2-digit',
    minute: '2-digit'
  })
}

// Fetch sections from the backend
const fetchSections = async () => {
  try {
    const response = await fetch('http://10.10.0.3:3001/api/sections')
    if (!response.ok) throw new Error('Failed to fetch sections')
    sections.value = await response.json()
  } catch (error) {
    console.error('Error fetching sections:', error)
    submitError.value = 'Failed to fetch sections'
  }
}

// Watch for changes in selectedSection to update building and classroom
watch(selectedSection, async (newSection) => {
  if (newSection) {
    try {
      const response = await fetch(`http://10.10.0.3:3001/api/sections/${newSection}`)
      if (!response.ok) throw new Error('Failed to fetch section details')
      const data = await response.json()
      building.value = data.classroom?.building?.name || ''
      classroom.value = data.classroom?.name || ''
      yearsection.value = data.name || ''
    } catch (error) {
      console.error('Error fetching section details:', error)
      submitError.value = 'Failed to fetch section details'
    }
  }
})

onMounted(() => {
  fetchSections()
  updateDateTime()
  setInterval(updateDateTime, 60000)
})

const handleSubmit = async () => {
  try {
    isSubmitting.value = true
    submitError.value = ''
    submitSuccess.value = false

    const formData = {
      professor_id: 1, // Replace with actual professor ID
      section_id: selectedSection.value,
      terminal_code: isUsingComputer.value ? terminalno.value : null, // Only include terminal if using computer
      student_full_name: name.value,
      student_email: email.value,
      student_number: parseInt(studentnumber.value),
      year_section: yearsection.value,
      remarks: remarks.value
    }

    const response = await fetch('http://10.10.0.3:3001/api/store/attendance', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify(formData)
    })

    if (!response.ok) {
      const errorData = await response.json()
      throw new Error(errorData.message || `Server error: ${response.status}`)
    }

    submitSuccess.value = true
    resetForm()
  } catch (error) {
    console.error('Error:', error)
    submitError.value = error.message || 'Failed to submit form'
  } finally {
    isSubmitting.value = false
  }
}

const resetForm = () => {
  subject.value = ''
  professorname.value = ''
  building.value = ''
  classroom.value = ''
  terminalno.value = ''
  name.value = ''
  email.value = ''
  studentnumber.value = ''
  yearsection.value = ''
  components.value = []
  remarks.value = ''
  selectedSection.value = ''
  isUsingComputer.value = false
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
        <span>Form submitted successfully!</span>
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
        <!-- Section Selection -->
        <div class="space-y-2">
          <label class="block text-sm font-medium text-gray-700">Section</label>
          <select v-model="selectedSection" required
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
            <option value="" disabled>Select Section</option>
            <option v-for="section in sections" :key="section.id" :value="section.id">
              {{ section.name }}
            </option>
          </select>
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

        <!-- Using Computer Checkbox -->
        <div class="space-y-2">
          <label class="flex items-center space-x-2 cursor-pointer">
            <input type="checkbox" v-model="isUsingComputer"
              class="rounded border-gray-300 text-blue-600 focus:ring-blue-500">
            <span class="text-sm font-medium text-gray-700">I am using a computer</span>
          </label>
        </div>

        <!-- Student Information -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div v-if="isUsingComputer" class="space-y-2 md:col-span-1">
            <label class="block text-sm font-medium text-gray-700">Terminal No.</label>
            <input type="text" v-model="terminalno" required
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
          </div>
          
          <div class="space-y-2 md:col-span-1">
            <label class="block text-sm font-medium text-gray-700">Full Name</label>
            <input type="text" v-model="name" required
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
          </div>
          
          <div class="space-y-2 md:col-span-1">
            <label class="block text-sm font-medium text-gray-700">Email</label>
            <input type="email" v-model="email" required
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
          </div>
          
          <div class="space-y-2 md:col-span-1">
            <label class="block text-sm font-medium text-gray-700">Student No</label>
            <input type="text" v-model="studentnumber" required
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
          </div>
        </div>

        <!-- Components Section - Only visible when using computer -->
        <div v-if="isUsingComputer" class="space-y-4">
          <h3 class="text-lg font-semibold text-gray-800">Condition:</h3>
          <p class="text-sm text-gray-600">
            Check if working properly, otherwise state the reason in remarks.
          </p>

          <!-- Select All Checkbox -->
          <div class="flex items-center mb-2">
            <label class="flex items-center space-x-2 cursor-pointer">
              <input type="checkbox" v-model="allComponentsSelected"
                class="rounded border-gray-300 text-blue-600 focus:ring-blue-500">
              <span class="text-sm font-medium text-gray-700">Select All</span>
            </label>
          </div>

          <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-5 gap-4">
            <label v-for="component in availableComponents" :key="component"
              class="flex items-center space-x-2 cursor-pointer">
              <input type="checkbox" :value="component" v-model="components"
                class="rounded border-gray-300 text-blue-600 focus:ring-blue-500">
              <span class="text-sm text-gray-700">{{ component.charAt(0).toUpperCase() + component.slice(1) }}</span>
            </label>
          </div>
        </div>

        <!-- Remarks Field -->
        <div class="space-y-2">
          <label class="block text-sm font-medium text-gray-700">Remarks:</label>
          <textarea v-model="remarks"
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 min-h-[80px]" />
        </div>

        <!-- Submit Button -->
        <div class="flex justify-center pt-4">
          <button type="submit" :disabled="isSubmitting"
            class="px-6 py-2 bg-blue-600 text-white rounded-lg shadow hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed transition-colors">
            {{ isSubmitting ? 'Submitting...' : 'Submit' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>