<script lang="ts" setup>
defineProps({
  modelValue: {
    type: String,
    required: true,
  },
  id: {
    type: String,
    default: "",
  },
  placeholder: {
    type: String,
    default: "",
  },
  customClasses: {
    type: String,
    default: "",
  },
});
const emit = defineEmits(["update:modelValue"]);
function handleInput(event: Event) {
  emit("update:modelValue", (event.target as HTMLInputElement).value);
}
</script>

<template>
  <input
    :id="id"
    :placeholder="placeholder"
    :value="modelValue"
    @input="handleInput"
    type="text"
    class="block w-full border-gray-300 rounded-md shadow-sm focus:border-primary-500 focus:ring-primary-500 sm:text-sm py-3"
    :class="customClasses"
  />
</template>
