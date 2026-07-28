<!-- Copyright (c) 2026, Ravindu Gajanayaka -->
<!-- Licensed under GPLv3. See license.txt -->

<script setup lang="ts">
import { ref, watch } from 'vue'
import { Delete } from 'lucide-vue-next'

const props = defineProps<{
  value: number
  activeMode?: 'qty' | 'discount' | 'price'
}>()

const emit = defineEmits<{
  'update:value': [value: number]
  'change-mode': [mode: 'qty' | 'discount' | 'price']
}>()

// Odoo maintains an active typing buffer string
const buffer = ref(props.value ? String(props.value) : '')
const mode = ref<'qty' | 'discount' | 'price'>(props.activeMode || 'qty')

// Sync buffer if selected item changes
watch(
  () => props.value,
  (newVal) => {
    buffer.value = newVal ? String(newVal) : ''
  }
)

function setMode(newMode: 'qty' | 'discount' | 'price') {
  mode.value = newMode
  buffer.value = '' // Clear buffer on mode switch, Odoo style
  emit('change-mode', newMode)
}

function handleInput(key: string) {
  if (key === 'DEL') {
    // Backspace action
    buffer.value = buffer.value.slice(0, -1)
  } else if (key === '+/-') {
    // Toggle negative/positive
    if (buffer.value.startsWith('-')) {
      buffer.value = buffer.value.slice(1)
    } else if (buffer.value !== '') {
      buffer.value = '-' + buffer.value
    }
  } else if (key === '.') {
    // Prevent multiple decimals
    if (!buffer.value.includes('.')) {
      buffer.value = buffer.value === '' ? '0.' : buffer.value + '.'
    }
  } else {
    // Append digit
    if (buffer.value === '0') {
      buffer.value = key
    } else {
      buffer.value += key
    }
  }

  // Parse string buffer to numeric value for cart update
  let parsed = parseFloat(buffer.value)

  // Standardize empty string or single decimal point to 0 for calculation
  if (isNaN(parsed) || buffer.value === '' || buffer.value === '.') {
    parsed = 0
  }

  // Emit update live without closing or clearing
  emit('update:value', parsed)
}
</script>

<template>
  <div class="w-full bg-gray-100 dark:bg-gray-800 p-2 rounded-xl">
    <!-- Action Modes (Qty / % / Price) -->
    <div class="grid grid-cols-4 gap-1 mb-1">
      <button 
        @click="setMode('qty')"
        class="h-12 font-bold rounded-lg border text-sm transition-colors"
        :class="mode === 'qty' ? 'bg-teal-600 text-white border-teal-600' : 'bg-white dark:bg-gray-700 border-gray-200 dark:border-gray-600'"
      >
        Qty
      </button>
      <button 
        @click="setMode('discount')"
        class="h-12 font-bold rounded-lg border text-sm transition-colors"
        :class="mode === 'discount' ? 'bg-teal-600 text-white border-teal-600' : 'bg-white dark:bg-gray-700 border-gray-200 dark:border-gray-600'"
      >
        %
      </button>
      <button 
        @click="setMode('price')"
        class="h-12 font-bold rounded-lg border text-sm transition-colors"
        :class="mode === 'price' ? 'bg-teal-600 text-white border-teal-600' : 'bg-white dark:bg-gray-700 border-gray-200 dark:border-gray-600'"
      >
        Price
      </button>
      <button 
        @click="handleInput('DEL')"
        class="h-12 flex items-center justify-center bg-red-400 text-white rounded-lg border border-red-400 active:bg-red-500"
      >
        <Delete :size="20" />
      </button>
    </div>

    <!-- Keypad Matrix -->
    <div class="grid grid-cols-3 gap-1">
      <button
        v-for="btn in ['1','2','3','4','5','6','7','8','9','+/-','0','.']"
        :key="btn"
        @click="handleInput(btn)"
        class="h-12 bg-white dark:bg-gray-700 text-gray-900 dark:text-gray-100 font-bold rounded-lg border border-gray-200 dark:border-gray-600 hover:bg-gray-50 active:bg-gray-200 text-lg flex items-center justify-center"
        :class="{ 'bg-amber-100 dark:bg-amber-900/40': btn === '+/-', 'bg-orange-100 dark:bg-orange-900/40': btn === '.' }"
      >
        {{ btn }}
      </button>
    </div>
  </div>
</template>
