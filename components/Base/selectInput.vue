<script setup lang="ts">
const props = defineProps({
  modelValue: {
    type: [String, Number],
    required: true,
  },
  id: {
    type: String,
    default: "",
  },
  customClasses: {
    type: String,
    default: "",
  },
  placeholder: {
    type: String,
    default: "select an option",
  },
});
const emit = defineEmits(["update:modelValue" , 'select']);
function handleInput(event :Event) {
  emit("update:modelValue", (event.target as HTMLInputElement).value)
}
watch(() => props.modelValue, (newValue) => {
  if (newValue) {
    emit("select", newValue);
  }
})

</script>

<template>
  <select
    :value="modelValue"
    @input="handleInput"
    class="block w-full border-gray-300 rounded-md shadow-sm focus:border-primary-500 focus:ring-primary-500 sm:text-sm px-3 py-3 mt-1 "
    :class="customClasses"
    :id="id"
  >
    <option value="" selected hidden class="sm:text-sm">
       {{placeholder }}
    </option>
    <slot></slot>
  </select>
</template>


