<script setup lang="ts">
import { storeToRefs } from "pinia";
import type { ComputedRef , Ref} from "vue";
import {
  ICategoryProperty,
  ISubCategory,
  ICategoryPropertyOption,
} from "@/types";
const otherStaticOption = {
  child: false,
  id: "other",
  name: "Other",
  slug: "other",
};
const formData = reactive({
  selected_main_category: "",
  selected_sub_category: "",
  props_values: [] as ICategoryProperty[],
});
const loading = ref(false);
const table: Ref<[value: string, name: string][]> = ref([]);
const {
  fetchMainCategories,
  fetchPropertiesByCategory,
  fetchChildOptionsForProperty,
} = useCategory();
const { getMainCategoriesList } = storeToRefs(useCategory());

onBeforeMount(async () => {
  loading.value = true;
  try {
    await fetchMainCategories();
  } finally {
    loading.value = false;
  }
});

const getSubCategory: ComputedRef<ISubCategory[] | undefined> = computed(() => {
  const mainCategory = getMainCategoriesList.value.find(
    (category) => category.id == formData.selected_main_category
  );

  return mainCategory?.children ? mainCategory.children : undefined;
});

const getProperties = async () => {
  loading.value = true;
  try {
    let properties = await fetchPropertiesByCategory(
      formData.selected_sub_category
    );
    formData.props_values = properties?.map((property) => {
      return {
        ...property,
        options: toRef([...property.options, otherStaticOption]),
      };
    });
  } finally {
    loading.value = false;
  }
};

const updateOption = (
  options: ICategoryPropertyOption[],
  newOptions: ICategoryPropertyOption[]
) => {
  let newOptionsWithOther = [...newOptions].map((option) => {
    return {
      ...option,
      options: [...option.options, otherStaticOption],
    };
  });
  return toRef(newOptionsWithOther);
};

const setChildOptions = (selectedOption, newOptions, props_values) => {
  // recursive function to set child options
  for (let props of props_values) {
    let optionIndex = props.options.findIndex(
      (option) => option.id == selectedOption.id
    );
    if (optionIndex != -1) {
      props.options[optionIndex] = {
        ...props.options[optionIndex],
        options: updateOption(props.options[optionIndex].options, newOptions),
      };
      break;
    } else {
      for (let option of props.options) {
        if (option.options?.length) {
          setChildOptions(selectedOption, newOptions, option.options);
        }
      }
    }
  }
};
const getChildOptions = async (selectedOption: ICategoryPropertyOption) => {
  loading.value = true;
  try {
    let newChildOptions = await fetchChildOptionsForProperty(selectedOption.id);
    setChildOptions(selectedOption, newChildOptions, formData.props_values);
  } finally {
    loading.value = false;
  }
};
const getRecursivekeyKeyBtValue = (propertyObject, value) => {
  console.log(propertyObject, value);
  for (let option of propertyObject.options) {
    if (option.id == value) {
      table.value.push({
        name: propertyObject.name,
        value: option.name,
      });
    }
    if (option.options?.length) {
      getRecursivekeyKeyBtValue(propertyObject, value);
    } else if (option.other_value == value) {
      table.value.push({
        name: propertyObject.name,
        value: option.other_value,
      });
    }
  }
};
const getSelectedOption = (property) => {
  if (property.other_value ) {
    table.value.push({
      name: property.name,
      value: property.other_value,
    });
  } else {
    for (let option of property.options) {
      if (option.id == property.value) {
        table.value.push({
          name: property.name,
          value: option.name,
        });
      } else if (option.other_value) {
        table.value.push({
          name: property.name,
          value: option.other_value,
        });
      }
      if (option.options?.length) {
        getSelectedOption(option);
      }
    }
  }
};
const generateTable = () => {
  for (let prop of formData.props_values) {
    if (prop.other_value ) {
      table.value.push({
        name: prop.name,
        value: prop.other_value,
      });
    } else {
      if (prop.value) {
        getSelectedOption(prop);
      }
    }
  }
  console.log(table.value);
};
</script>

<template>
  <!-- center -->
  <div class="flex justify-center items-center py-20 w-full">
    <!-- loader -->
    <base-loader v-if="loading" />
    <div v-else>
      <form @submit.prevent="generateTable">
        <div class="flex flex-col justify-center items-center">
          <h1 class="text-3xl font-bold mb-5">Form</h1>
          <p class="text-gray-500 mb-5">Please fill out the form below</p>
        </div>
        <div class="grid grid-cols-12 gap-2 justify-items-center px-12">
          <div
            class="col-span-12 grid grid-cols-2 gap-2 justify-items-center max-w-xl w-full"
          >
            <div :class="{ '!col-span-2': !getSubCategory?.length }">
              <base-select-input
                v-model="formData.selected_main_category"
                @select="
                  formData.selected_sub_category = '';
                  formData.props_values = [];
                "
              >
                <option
                  v-for="category in getMainCategoriesList"
                  :key="category.id"
                  :value="category.id"
                >
                  {{ category.name }}
                </option>
              </base-select-input>
            </div>
            <div v-if="getSubCategory?.length" class="w-full">
              <base-select-input
                v-model="formData.selected_sub_category"
                placeholder="Select a sub category"
                @select="getProperties"
              >
                <option
                  v-for="category in getSubCategory"
                  :key="category.id"
                  :value="category.id"
                >
                  {{ category.name }}
                </option>
              </base-select-input>
            </div>
          </div>
          <div
            class="col-span-12 gap-2 justify-items-center max-w-xl w-full"
            v-if="formData.props_values.length"
          >
            <category-recursive-options
              v-for="property in formData.props_values"
              :key="property.id + 'recursive'"
              :property="property"
              @select-option="getChildOptions"
            />
          </div>
        </div>
        <div class="flex justify-center items-center mt-10">
          <base-button class="w-fit px-5 py-3" type="submit"
            >Submit</base-button
          >
        </div>
      </form>
      <div class="mt-5">
        <base-table :data="table.filter(el=>el.value)" />
      </div>
    </div>
  </div>
</template>
