<script setup>
defineProps({
  data: {
    type: Object,
    default: null,
  },
});
</script>

<template>
    <div class="flex flex-col border border-green-800">
        <div class="flex border border-green-800">
          <div v-for="(value, key) in data" :key="key" class="flex-1 text-center border border-green-800 py-2">
            <strong>{{ value.name }}</strong>
          </div>
        </div>
      
        <div class="flex">
          <div v-for="(value, key) in data" :key="key" class="flex-1 text-center border border-green-800 py-2">
            {{ value ? value.value : "null" }}
          </div>
        </div>
      </div>
      
</template>
