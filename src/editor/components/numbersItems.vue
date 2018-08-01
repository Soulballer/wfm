<template>
  <div>
    <numbers-item
      v-for="number in numbers"

      :currentFlowDeployedData="currentFlowDeployedData"
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
        type: Object,
        default() {
          return {}
        }
      },
      numbers: {
        type: Array,
        default() {
          return []
        }
      },
      readonly: {
        type : Boolean,
        default() {
          return false
        }
      }
    },
    components : { NumbersItem }
  }
</script>

<style lang="scss" scoped>

</style>
