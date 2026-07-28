<!-- Copyright (c) 2026, Ravindu Gajanayaka -->
<!-- Licensed under GPLv3. See license.txt -->

<script setup lang="ts">
import { ref, watch } from 'vue'
import { Delete } from 'lucide-vue-next'

const props = defineProps<{
  value: number
  label?: string
}>()

const emit = defineEmits<{
  'update:value': [value: number]
  close: []
}>()

// Internal string buffer to accumulate keypresses (like Odoo POS)
const buffer = ref<string>('')
const isBufferFresh = ref<boolean>(true)

// Sync when selected item or initial value changes
watch(
  () => props.value,
  (newVal) => {
    buffer.value = String(newVal)
    isBufferFresh.value = true
  },
  { immediate: true }
)

const numpadKeys = [
  '1', '2', '3',
  '4', '5', '6',
  '7', '8', '9',
  '+/-', '0', '.'
]

function handleKeyPress(key: string) {
  // If starting a new key entry session, reset buffer
  if (isBufferFresh.value && key !== 'DEL') {
    buffer.value = ''
    isBufferFresh.value = false
  }

  if (key === 'DEL') {
    buffer.value = buffer.value.slice(0, -1)
  } else if (key === '.') {
    if (!buffer.value.includes('.')) {
      // Odoo logic: starting with '.' turns into '0.'
      buffer.value = buffer.value ? buffer.value + '.' : '0.'
    }
  } else if (key === '+/-') {
    if (buffer.value.startsWith('-')) {
      buffer.value = buffer.value.slice(1)
    } else if (buffer.value && buffer.value !== '0') {
      buffer.value = '-' + buffer.value
    }
  } else {
    // Digit keypress
    if (buffer.value === '0') {
      buffer.value = key
    } else {
      buffer.value += key
    }
  }

  commitUpdate()
}

function commitUpdate() {
  // If buffer ends in '.' or is empty, wait for next keypress before updating store
  if (!buffer.value || buffer.value.endsWith('.')) return

  const parsed = parseFloat(buffer.value)
  if (!isNaN(parsed) && parsed > 0) {
    emit('update:value', parsed)
  }
}

function handleBackspace() {
  handleKeyPress('DEL')
}
</script>

<template>
  <div class="bg-gray-100 dark:bg-gray-800 p-2 rounded-xl select-none">
    <!-- Active Buffer Display Header -->
    <div class="bg-white dark:bg-gray-900 border border-gray-200 dark:border-gray-700 rounded-lg p-2 mb-2 text-right">
      <span class="text-xs text-gray-400 block uppercase font-semibold">{{ label || 'Quantity' }}</span>
      <span class="text-2xl font-bold text-gray-900 dark:text-gray-100 font-mono">
        {{ buffer || '0' }}
      </span>
    </div>

    <!-- Keypad Grid -->
    <div class="grid grid-cols-4 gap-1.5">
      <!-- 1-9, 0, +/-, . Buttons -->
      <div class="col-span-3 grid grid-cols-3 gap-1.5">
        <button
          v-for="k in numpadKeys"
          :key="k"
          @click="handleKeyPress(k)"
          class="h-12 rounded-lg font-bold text-lg transition-all active:scale-95 flex items-center justify-center border"
          :class="
            k === '+/-'
              ? 'bg-amber-100 dark:bg-amber-900/40 text-amber-800 dark:text-amber-200 border-amber-200 dark:border-amber-800'
              : k === '.'
                ? 'bg-orange-100 dark:bg-orange-900/40 text-orange-800 dark:text-orange-200 border-orange-200 dark:border-orange-800'
                : 'bg-white dark:bg-gray-900 text-gray-800 dark:text-gray-100 border-gray-200 dark:border-gray-700 hover:bg-gray-50'
          "
        >
          {{ k }}
        </button>
      </div>

      <!-- Action Column (Backspace / Mode Control) -->
      <div class="flex flex-col gap-1.5">
        <button
          @click="handleBackspace"
          aria-label="Backspace"
          class="h-12 bg-red-100 dark:bg-red-900/40 text-red-600 dark:text-red-300 border border-red-200 dark:border-red-800 rounded-lg flex items-center justify-center active:scale-95 transition-all"
        >
          <Delete :size="20" />
        </button>
        <button
          class="flex-1 bg-teal-500 text-white font-bold text-sm rounded-lg flex items-center justify-center border border-teal-600 shadow-sm"
        >
          Qty
        </button>
      </div>
    </div>
  </div>
</template>
