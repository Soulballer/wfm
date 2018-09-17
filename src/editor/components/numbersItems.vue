<template>
  <div>
    <numbers-item
      v-for="number in numbers"

      :isAdmin="isAdmin"
      :isLoading="isLoading"
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
      isAdmin: {
        type: Boolean,
        default() {
          return false
        }
      },
      isLoading: {
        type: Boolean,
        default() {
          return false
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
