<script setup>
import { ref, onMounted, watch } from 'vue';
import AlertMessage from './AlertMessage.vue';
import FormField from './FormField.vue';
import ComponentCheckbox from './Checkbox.vue';

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
const number = ref('')
const components = ref([])
const remarks = ref('')
const sections = ref([])
const selectedSection = ref('')

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
    const response = await fetch('http://localhost:8000/api/sections')
    if (!response.ok) throw new Error('Failed to fetch sections')
    sections.value = await response.json()
  } catch (error) {
    console.error('Error fetching sections:', error)
  }
}

// Watch for changes in selectedSection to update building and classroom
watch(selectedSection, async (newSection) => {
  if (newSection) {
    try {
      const response = await fetch(`http://localhost:8000/api/sections/${newSection}`)
      if (!response.ok) throw new Error('Failed to fetch building and classroom')
      const data = await response.json()
      building.value = data.building
      classroom.value = data.classroom
    } catch (error) {
      console.error('Error fetching building and classroom:', error)
    }
  }
})

// Initialize date and time on component mount
onMounted(() => {
  fetchSections()
  updateDateTime()
  // Update time every minute
  setInterval(updateDateTime, 60000)
})

const handleSubmit = async () => {
  try {
    isSubmitting.value = true
    submitError.value = ''
    submitSuccess.value = false

    // Validate required fields
    const requiredFields = {
      subject: subject.value,
      professorname: professorname.value,
      building: building.value,
      classroom: classroom.value,
      terminalno: terminalno.value,
      name: name.value,
      email: email.value,
      studentnumber: studentnumber.value,
      yearsection: yearsection.value
    }

    // Check for empty required fields
    const emptyFields = Object.entries(requiredFields)
      .filter(([_, value]) => !value.trim())
      .map(([key]) => key)

    if (emptyFields.length > 0) {
      throw new Error(`Please fill in all required fields: ${emptyFields.join(', ')}`)
    }

    // Create form data object
    const formData = {
      subject: subject.value,
      professorname: professorname.value,
      building: building.value,
      classroom: classroom.value,
      terminalno: terminalno.value,
      name: name.value,
      email: email.value,
      studentnumber: studentnumber.value,
      yearsection: yearsection.value,
      number: number.value,
      components: components.value,
      remarks: remarks.value
    }

    const response = await fetch('https://qcu-rest-api-attendance.vercel.app/api/create', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type',
        'Access-Control-Allow-Methods': 'POST, GET, OPTIONS',
      },
      body: JSON.stringify(formData)
    })

    if (!response.ok) {
      const errorData = await response.json().catch(() => null)
      throw new Error(errorData?.message || `Server error: ${response.status}`)
    }

    const data = await response.json()
    console.log('Success:', data)
    submitSuccess.value = true

    // Reset form
    resetForm()

  } catch (error) {
    console.error('Error:', error)
    submitError.value = error.message || 'Failed to submit form. Please try again.'
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
  number.value = ''
  components.value = []
  remarks.value = ''
}
</script>

<template>
  <div class="min-h-screen bg-gray-50 p-4 md:p-6 lg:p-8">
    <div class="mx-auto max-w-4xl bg-white rounded-lg shadow-md p-4 md:p-6">
      <h1 class="text-2xl md:text-3xl font-bold text-gray-800 mb-6 text-center">
        Utilization Monitoring Form
      </h1>

      <AlertMessage v-if="submitSuccess" type="success" message="Form submitted successfully!" />

      <AlertMessage v-if="submitError" type="error" :message="submitError" />

      <form @submit.prevent="handleSubmit" class="space-y-6">
        <!-- First Section -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <FormField label="Name of Faculty" v-model="professorname" required />
          <FormField label="Subject/Code" v-model="subject" required />
        </div>

        <hr class="border-gray-200" />

        <!-- Second Section -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div class="space-y-4">
            <FormField label="Terminal No." v-model="terminalno" required />
            <FormField label="Full Name" v-model="name" required />
            <FormField label="Email" v-model="email" type="email" required />
          </div>

          <div class="space-y-4">
            <FormField label="Student No" v-model="studentnumber" required />
            <div class="space-y-2">
              <label class="block text-sm font-medium text-gray-700">Year and Section</label>
              <select v-model="selectedSection" required
                class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                <option value="" disabled>Select Section</option>
                <option v-for="section in sections" :key="section.id" :value="section.id">
                  {{ section.name }}
                </option>
              </select>
            </div>

            <div class="grid grid-cols-2 gap-4">
              <FormField label="Building" v-model="building" readonly />
              <FormField label="Classroom" v-model="classroom" readonly />
            </div>
          </div>
        </div>

        <hr class="border-gray-200" />

        <!-- Components Section -->
        <div class="space-y-4">
          <h3 class="text-lg font-semibold text-gray-800">Condition:</h3>
          <p class="text-sm text-gray-600">
            Check if working properly, otherwise state the reason in remarks.
          </p>

          <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-5 gap-4">
            <ComponentCheckbox v-for="component in [ 'System Unit', 'Monitor', 'Keyboard', 'Mouse', 'Network' ]"
              :key="component" :label="component" v-model="components" :value="component.toLowerCase()" />
          </div>

          <div class="space-y-2">
            <label class="block text-sm font-medium text-gray-700">Remarks:</label>
            <textarea v-model="remarks"
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 min-h-[80px]" />
          </div>
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