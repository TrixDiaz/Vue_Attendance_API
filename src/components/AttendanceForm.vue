<script setup>
import { ref, onMounted } from 'vue'

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

// Initialize date and time on component mount
onMounted(() => {
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
      date: date.value,
      day: day.value,
      time: time.value,
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

    // Make POST request
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

    // Reset form after successful submission
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

  } catch (error) {
    console.error('Error:', error)
    submitError.value = error.message || 'Failed to submit form. Please try again.'
  } finally {
    isSubmitting.value = false
  }
}
</script>

<template>
  <div class="min-h-screen bg-gray-50 p-4 md:p-6 lg:p-8">
    <div class="mx-auto max-w-4xl bg-white rounded-lg shadow-md p-6">
      <h1 class="text-2xl md:text-3xl font-bold text-gray-800 mb-6 text-center">
        Utilization Monitoring Form
      </h1>

      <!-- Messages -->
      <div v-if="submitSuccess" class="mb-4 p-4 bg-green-100 text-green-700 rounded-md">
        Form submitted successfully!
      </div>

      <div v-if="submitError" class="mb-4 p-4 bg-red-100 text-red-700 rounded-md">
        {{ submitError }}
      </div>

      <!-- Form content remains the same as in your original template -->
      <form @submit.prevent="handleSubmit" class="space-y-6">
        <!-- Rest of the form template remains the same -->
        <!-- First Section -->
        <div class="space-y-4">
          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div class="space-y-4">
              <div class="space-y-2">
                <label for="date" class="block text-sm font-medium text-gray-700">Date:</label>
                <input type="date" v-model="date" required readonly
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>

              <div class="space-y-2">
                <label for="day" class="block text-sm font-medium text-gray-700">Day:</label>
                <input type="text" v-model="day" required readonly
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>

              <div class="space-y-2">
                <label for="time" class="block text-sm font-medium text-gray-700">Time:</label>
                <input type="time" v-model="time" required readonly
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>

              <div class="space-y-2">
                <label for="subject" class="block text-sm font-medium text-gray-700">Subject/Code:</label>
                <input type="text" v-model="subject" required
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>
            </div>

            <div class="space-y-4">
              <div class="space-y-2">
                <label for="professorname" class="block text-sm font-medium text-gray-700">Name of Faculty:</label>
                <input type="text" v-model="professorname" required
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>

              <div class="space-y-2">
                <label for="building" class="block text-sm font-medium text-gray-700">Building:</label>
                <input type="text" v-model="building" required
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>

              <div class="space-y-2">
                <label for="classroom" class="block text-sm font-medium text-gray-700">Room:</label>
                <input type="text" v-model="classroom" required
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>
            </div>
          </div>
        </div>

        <hr class="border-gray-200" />

        <!-- Second Section -->
        <div class="space-y-4">
          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div class="space-y-4">
              <div class="space-y-2">
                <label for="terminalno" class="block text-sm font-medium text-gray-700">Terminal No.:</label>
                <input type="text" v-model="terminalno" required
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>

              <div class="space-y-2">
                <label for="name" class="block text-sm font-medium text-gray-700">Full Name:</label>
                <input type="text" v-model="name" required
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>

              <div class="space-y-2">
                <label for="email" class="block text-sm font-medium text-gray-700">Email:</label>
                <input type="email" v-model="email" required
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>
            </div>

            <div class="space-y-4">
              <div class="space-y-2">
                <label for="studentnumber" class="block text-sm font-medium text-gray-700">Student No:</label>
                <input type="text" v-model="studentnumber" required
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>

              <div class="space-y-2">
                <label for="yearsection" class="block text-sm font-medium text-gray-700">Year and Section:</label>
                <input type="text" v-model="yearsection" required
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>

              <div class="space-y-2">
                <label for="number" class="block text-sm font-medium text-gray-700">Number:</label>
                <input type="text" v-model="number"
                       class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
              </div>
            </div>
          </div>
        </div>

        <hr class="border-gray-200" />

        <!-- Components Section -->
        <div class="space-y-4">
          <h3 class="text-lg font-semibold text-gray-800">Condition:</h3>
          <p class="text-sm text-gray-600">
            Check if working properly, otherwise don't check and state the reason in the remarks.
          </p>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div class="space-y-3">
              <div class="flex flex-wrap gap-4">
                <label class="flex items-center space-x-2">
                  <input type="checkbox" v-model="components" value="system_unit"
                         class="rounded border-gray-300 text-blue-600 focus:ring-blue-500" />
                  <span class="text-sm text-gray-700">System Unit</span>
                </label>
                <label class="flex items-center space-x-2">
                  <input type="checkbox" v-model="components" value="monitor"
                         class="rounded border-gray-300 text-blue-600 focus:ring-blue-500" />
                  <span class="text-sm text-gray-700">Monitor</span>
                </label>
                <label class="flex items-center space-x-2">
                  <input type="checkbox" v-model="components" value="keyboard"
                         class="rounded border-gray-300 text-blue-600 focus:ring-blue-500" />
                  <span class="text-sm text-gray-700">Keyboard</span>
                </label>
                <label class="flex items-center space-x-2">
                  <input type="checkbox" v-model="components" value="mouse"
                         class="rounded border-gray-300 text-blue-600 focus:ring-blue-500" />
                  <span class="text-sm text-gray-700">Mouse</span>
                </label>
                <label class="flex items-center space-x-2">
                  <input type="checkbox" v-model="components" value="network"
                         class="rounded border-gray-300 text-blue-600 focus:ring-blue-500" />
                  <span class="text-sm text-gray-700">Network</span>
                </label>
              </div>
            </div>

            <div class="space-y-2">
              <label for="remarks" class="block text-sm font-medium text-gray-700">Remarks:</label>
              <input type="text" v-model="remarks"
                     class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500" />
            </div>
          </div>
        </div>

        <!-- Submit Button -->
        <div class="flex justify-center pt-4">
          <button type="submit" :disabled="isSubmitting"
                  class="px-6 py-2 bg-blue-600 text-white rounded-md shadow-sm hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed">
            {{ isSubmitting ? 'Submitting...' : 'Submit' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>