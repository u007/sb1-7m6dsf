<script setup lang="ts">
import { ref } from 'vue'
import FunctionForm from './components/FunctionForm.vue'

const functionData = ref({
  name: '',
  description: '',
  parameters: {
    type: 'object',
    properties: {},
    required: []
  }
})

const handleSubmit = () => {
  console.log('Function data:', JSON.stringify(functionData.value, null, 2))
}
</script>

<template>
  <div class="min-h-screen bg-gray-100 py-6 flex flex-col justify-center sm:py-12">
    <div class="relative py-3 sm:max-w-xl sm:mx-auto">
      <div class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20">
        <h1 class="text-3xl font-bold mb-6 text-center text-gray-900">OpenAI Function Call Parameter Form</h1>
        <FunctionForm v-model="functionData" @submit="handleSubmit" />
      </div>
    </div>
  </div>
</template>