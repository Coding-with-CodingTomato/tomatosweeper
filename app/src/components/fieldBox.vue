<template>
  <div :style="{ backgroundColor: !props.revealed ? 'grey' : 'darkgrey' }">
    <div class="content" v-if="props.revealed">
      <img v-if="props.tomato" src="../assets/pepe_angry250.png" />
      <span v-if="!props.tomato && props.neighbors !== 0">{{ neighbors }}</span>
    </div>
    <div class="content" v-if="!props.revealed">
      <img class="flag" v-if="props.flagged" src="../assets/flag.svg" />
      <span>{{ row }}, {{ col }}</span>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  row: Number,
  col: Number,
  revealed: Boolean,
  flagged: Boolean,
  tomato: Boolean,
  neighbors: Number,
});
</script>

<style scoped>
.content {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
img {
  width: 100%;
}

.flag {
  width: 50%;
}
</style>
