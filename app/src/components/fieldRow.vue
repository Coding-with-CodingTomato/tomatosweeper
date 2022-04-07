<template>
  <fieldBox
    v-for="box in props.row"
    :key="box"
    :="box"
    @click="$emit('reveal', box.row, box.col)"
  />
</template>

<script setup>
import fieldBox from "./fieldBox.vue";

const props = defineProps({
  row: Array,
});

defineEmits(["reveal"]);
</script>
