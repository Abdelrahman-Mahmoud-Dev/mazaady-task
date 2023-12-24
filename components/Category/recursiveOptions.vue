<script setup lang="ts">
defineProps({
  property: {
    type: Object,
    default: null,
  },
});
const emit = defineEmits(["select-option"]);
const hasChildern = (option) => {
  return option.child;
};
const getChildOptions = async (prop) => {
  let optionIndex = prop?.options?.findIndex(
    (option) => option.id == prop.value
  );
  if (hasChildern(prop)) {
    emit("select-option", prop);
  } else if (hasChildern(prop.options[optionIndex])) {
    emit("select-option", prop.options[optionIndex]);
  }
};
</script>
<template>
  <label class="text-gray-500">{{ property.name }}</label>
  <base-select-input
    class="col-span-6 w-full"
    v-model="property.value"
    :placeholder="`Select ${property.name}`"
    @select="getChildOptions(property)"
  >
    <option
      v-for="option in property.options"
      :key="option.id"
      :value="option.id"
    >
      {{ option.name }}
    </option>
  </base-select-input>
  <base-text-input
    :key="property.id + 'other'"
    v-if="property.value == 'other'"
    v-model="property.other_value"
    :placeholder="`write value for ${property.name}`"
    customClasses="border-red-500 mt-5"
  >
  </base-text-input>

  <div
    v-if="
      property?.options?.[
        property?.options?.findIndex((option) => option.id == property.value)
      ]?.options?.length
    "
  >
    <category-recursive-options
      v-for="prop in property.options[
        property.options.findIndex((option) => option.id == property.value)
      ]?.options"
      :property="prop"
      :key="prop.id + 'recursive'"
      @select-option="getChildOptions"
    />
  </div>
</template>
