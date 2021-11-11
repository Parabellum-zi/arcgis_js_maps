<template>
  <div id="MapPopup">
    <p>{{props.cur}}</p>
    <p>{{cur}}</p>
  </div>
</template>

<script setup>
import {ref, onMounted,defineProps} from "vue";

const props = defineProps({
  cur: {
    type: Object,
    // default: () => {}
  },
})

</script>

<style scoped>
  #MapPopup{
    height:250px;
    width:430px;
  }
</style>