<template>
  <div>
    <numbers-item
      v-for="number in numbers"
      key="number.name"
      :currentFlowDeployedData="currentFlowDeployedData"
      :number="number"
      :isData="isData"
      :readonly="readonly"
    ></numbers-item>
  </div>
</template>

<script>
import NumbersItem from './NumbersItem'

export default {
  name: 'NumbersItems',
  components : {NumbersItem},

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
}
</script>

<style lang="scss" scoped>

</style>
