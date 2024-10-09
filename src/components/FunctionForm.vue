<script setup lang="ts">
import { ref, watch } from 'vue'

const props = defineProps<{
  modelValue: {
    name: string;
    description: string;
    service: string;
    parameters: {
      type: string;
      properties: Record<string, any>;
      required: string[];
    };
  };
}>();

const emit = defineEmits(['update:modelValue', 'submit']);

const localValue = ref({
  name: '',
  description: '',
  service: '',
  parameters: {
    type: 'object',
    properties: {},
    required: []
  }
});

// List of available services
const services = [
  'OpenAI',
  'Google Cloud',
  'AWS',
  'Azure',
  'Custom Endpoint'
];

// Initialize localValue with props.modelValue or default values
watch(() => props.modelValue, (newValue) => {
  if (newValue) {
    localValue.value = {
      name: newValue.name || '',
      description: newValue.description || '',
      service: newValue.service || '',
      parameters: {
        type: newValue.parameters?.type || 'object',
        properties: newValue.parameters?.properties || {},
        required: newValue.parameters?.required || []
      }
    };
  }
}, { immediate: true });

const addProperty = () => {
  const newPropertyName = `property${Object.keys(localValue.value.parameters.properties).length + 1}`;
  localValue.value.parameters.properties[newPropertyName] = {
    type: 'string',
    description: ''
  };
  emit('update:modelValue', localValue.value);
};

const removeProperty = (propertyName: string) => {
  if (confirm(`Are you sure you want to remove the property "${propertyName}"?`)) {
    delete localValue.value.parameters.properties[propertyName];
    localValue.value.parameters.required = localValue.value.parameters.required.filter(name => name !== propertyName);
    emit('update:modelValue', localValue.value);
  }
};

const updateRequired = (propertyName: string, isRequired: boolean) => {
  if (isRequired) {
    if (!localValue.value.parameters.required.includes(propertyName)) {
      localValue.value.parameters.required.push(propertyName);
    }
  } else {
    localValue.value.parameters.required = localValue.value.parameters.required.filter(name => name !== propertyName);
  }
  emit('update:modelValue', localValue.value);
};

const handleSubmit = () => {
  emit('submit', localValue.value);
};
</script>

<template>
  <form @submit.prevent="handleSubmit" class="space-y-6">
    <div>
      <label for="function-name" class="block text-sm font-medium text-gray-700">Function Name:</label>
      <input
        id="function-name"
        v-model="localValue.name"
        type="text"
        required
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 px-4 py-2"
      />
    </div>

    <div>
      <label for="function-description" class="block text-sm font-medium text-gray-700">Function Description:</label>
      <textarea
        id="function-description"
        v-model="localValue.description"
        required
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 px-4 py-2"
      ></textarea>
    </div>

    <div>
      <label for="function-service" class="block text-sm font-medium text-gray-700">Function Service Endpoint:</label>
      <select
        id="function-service"
        v-model="localValue.service"
        required
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 px-4 py-2"
      >
        <option value="" disabled>Select a service</option>
        <option v-for="service in services" :key="service" :value="service">
          {{ service }}
        </option>
      </select>
    </div>

    <div>
      <h3 class="text-lg font-medium text-gray-900">Parameters</h3>
      <div v-if="localValue.parameters && localValue.parameters.properties">
        <div v-for="(property, name) in localValue.parameters.properties" :key="name" class="mt-4 p-4 bg-gray-50 rounded-md relative">
          <button
            @click="removeProperty(name)"
            class="absolute top-2 right-2 text-red-600 hover:text-red-800"
            type="button"
          >
            &#x2715;
          </button>
          <div>
            <label :for="`property-name-${name}`" class="block text-sm font-medium text-gray-700">Property Name:</label>
            <input
              :id="`property-name-${name}`"
              v-model="localValue.parameters.properties[name].name"
              type="text"
              required
              class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 px-4 py-2"
            />
          </div>
          <div class="mt-2">
            <label :for="`property-type-${name}`" class="block text-sm font-medium text-gray-700">Type:</label>
            <select
              :id="`property-type-${name}`"
              v-model="localValue.parameters.properties[name].type"
              class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 px-4 py-2"
            >
              <option value="string">String</option>
              <option value="number">Number</option>
              <option value="boolean">Boolean</option>
              <option value="object">Object</option>
              <option value="array">Array</option>
            </select>
          </div>
          <div class="mt-2">
            <label :for="`property-description-${name}`" class="block text-sm font-medium text-gray-700">Description:</label>
            <textarea
              :id="`property-description-${name}`"
              v-model="localValue.parameters.properties[name].description"
              class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50 px-4 py-2"
            ></textarea>
          </div>
          <div class="mt-2">
            <label class="inline-flex items-center">
              <input
                type="checkbox"
                :checked="localValue.parameters.required.includes(name)"
                @change="updateRequired(name, $event.target.checked)"
                class="rounded border-gray-300 text-indigo-600 shadow-sm focus:border-indigo-300 focus:ring focus:ring-offset-0 focus:ring-indigo-200 focus:ring-opacity-50"
              />
              <span class="ml-2 text-sm text-gray-700">Required</span>
            </label>
          </div>
        </div>
      </div>
      <button
        type="button"
        @click="addProperty"
        class="mt-4 inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        Add Property
      </button>
    </div>

    <div>
      <button
        type="submit"
        class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        Submit
      </button>
    </div>
  </form>
</template>