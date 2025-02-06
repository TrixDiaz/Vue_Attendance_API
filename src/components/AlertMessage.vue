<!-- components/AlertMessage.vue -->
<script setup>
defineProps({
    type: {
        type: String,
        required: true,
        validator: (value) => ['success', 'error'].includes(value)
    },
    message: {
        type: String,
        required: true
    }
})
</script>

<template>
  <div
    :class="[
      'mb-4 p-4 rounded-lg flex items-center gap-2',
      type === 'success' ? 'bg-green-50 text-green-700' : 'bg-red-50 text-red-700'
    ]"
  >
    <svg
      v-if="type === 'success'"
      xmlns="http://www.w3.org/2000/svg"
      class="h-5 w-5"
      viewBox="0 0 20 20"
      fill="currentColor"
    >
      <path
        fill-rule="evenodd"
        d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"
        clip-rule="evenodd"
      />
    </svg>
    <svg
      v-else
      xmlns="http://www.w3.org/2000/svg"
      class="h-5 w-5"
      viewBox="0 0 20 20"
      fill="currentColor"
    >
      <path
        fill-rule="evenodd"
        d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7 4a1 1 0 11-2 0 1 1 0 012 0zm-1-9a1 1 0 00-1 1v4a1 1 0 102 0V6a1 1 0 00-1-1z"
        clip-rule="evenodd"
      />
    </svg>
    <span>{{ message }}</span>
  </div>
</template>
