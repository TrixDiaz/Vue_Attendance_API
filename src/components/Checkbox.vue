<script setup>
import {computed} from 'vue';

const props = defineProps({
  label: {
    type: String,
    required: true
  },
  modelValue: {
    type: Array,
    required: true
  },
  value: {
    type: String,
    required: true
  }
});

const emit = defineEmits(['update:modelValue']);

const isChecked = computed(() => {
  return props.modelValue.includes(props.value);
});

const updateCheckbox = (event) => {
  const isChecked = event.target.checked;
  const newValue = [...props.modelValue];

  if (isChecked) {
    newValue.push(props.value);
  } else {
    const index = newValue.indexOf(props.value);
    if (index > -1) {
      newValue.splice(index, 1);
    }
  }

  emit('update:modelValue', newValue);
};
</script>

<template>
  <label class="flex items-center space-x-2 cursor-pointer">
    <input
        type="checkbox"
        :checked="isChecked"
        @change="updateCheckbox"
        class="rounded border-gray-300 text-blue-600 focus:ring-blue-500"
    />
    <span class="text-sm text-gray-700">{{ label }}</span>
  </label>
</template>