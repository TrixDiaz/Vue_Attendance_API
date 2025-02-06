<script setup>
defineProps({
    label: {
        type: String,
        required: true
    },
    modelValue: {
        type: [String, Number],
        default: ''
    },
    type: {
        type: String,
        default: 'text'
    },
    required: {
        type: Boolean,
        default: false
    },
    readonly: {
        type: Boolean,
        default: false
    },
    placeholder: {
        type: String,
        default: ''
    },
    error: {
        type: String,
        default: ''
    }
})

defineEmits(['update:modelValue'])
</script>

<template>
    <div class="space-y-2">
        <label class="block text-sm font-medium text-gray-700">
            {{ label }}
            <span v-if="required" class="text-red-500">*</span>
        </label>
        <input :type="type" :value="modelValue" @input="$emit('update:modelValue', $event.target.value)"
            :required="required" :readonly="readonly" :placeholder="placeholder" :class="[
                'w-full px-3 py-2 border border-gray-300 rounded-lg',
                'focus:ring-2 focus:ring-blue-500 focus:border-blue-500',
                'transition-colors duration-200',
                { 'bg-gray-50 cursor-not-allowed': readonly },
                { 'border-red-500': error }
            ]" />
        <p v-if="error" class="text-sm text-red-500">{{ error }}</p>
    </div>
</template>