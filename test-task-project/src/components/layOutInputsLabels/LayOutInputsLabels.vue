<template>
  <div class="container">
    <input
      type="text"
      v-model="inputValue"
      @input="updateValue"
      :placeholder="props.placeholder"
    />
    <label>{{ displayValue ? displayValue : "0" }}</label>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, watch } from "vue";

export default defineComponent({
  props: {
    placeholder: String,
    id: String,
    modelValue: {
      type: Number,
      required: true,
    },
  },
  setup(props, { emit }) {
    const inputValue = ref(props.modelValue);
    const displayValue = ref(props.modelValue);
    let debounceTimer: number | null = null;

    const updateValue = () => {
      if (debounceTimer) {
        clearTimeout(debounceTimer);
      }
      inputValue.value = +inputValue.value.toString().replace(/[^\d.]/g, "");

      debounceTimer = setTimeout(() => {
        displayValue.value = inputValue.value;
        emit("update:modelValue", inputValue.value);
      }, 300);
    };

    watch(
      () => props.modelValue,
      (newValue) => {
        inputValue.value = newValue;
        displayValue.value = newValue;
      }
    );
    return { inputValue, updateValue, props, displayValue };
  },
  components: {},
});
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  width: 100%;
  gap: 16px;
  > input {
    outline: none;
  }
}
</style>
