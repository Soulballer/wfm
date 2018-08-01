<template>
  <div>
    <numbers-item
      v-for="number in numbers"

      :currentFlowDeployedData="currentFlowDeployedData"
      :isData="isData"
      :key="number.name"
      :number="number"
      :readonly="readonly"
    ></numbers-item>
  </div>
</template>

<script>
import NumbersItem from './numbersItem.vue'

export default {
  props: {
    currentFlowDeployedData: {
      type: Boolean
    },
    numbers: {
      type: Array
    },
    isData: {
      type: Boolean
    },
    readonly: {
      type : Boolean
    },
  },
  components : { NumbersItem }
}
</script>

<style lang="scss" scoped>

</style>
