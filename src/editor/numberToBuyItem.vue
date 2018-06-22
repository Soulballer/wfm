<template>
  <or-checkbox
  :value="isInBuyList"
  @change="updateBuyList">
    <div class="number-to-buy-item">
      <span>{{number.phoneNumber}}</span>
      <span>{{getState(number.region) ? getState(number.region).name : null}}</span>
      <!-- <span>$ {{number.price}} monthly</span> -->
    </div>
  </or-checkbox>
</template>

<script>
import eventHub from './eventHub.js';

export default {
  name: 'NumbersToBuyItem',
  computed: {
    isInBuyList () {
      return _.find(this.buyList, x => x.phoneNumber === this.number.phoneNumber) !== undefined;
    }
  },
  methods: {
    getState (region) {
      return _.find(this.states, s => s.value === region);
    },
    updateBuyList (event) {
      eventHub.$emit('buyList:update',
      event ? _.concat(this.buyList, this.number)
            : _.reject(this.buyList, this.number))
    }
  },
  props: {
    number: {
      type: Object
    },
    buyList: {
      type: Array
    },
    states: {
      type: Array
    }
  }
}
</script>

<style lang='scss' scoped>
  .number-to-buy-item {
    display: flex;
    justify-content: space-between;
  }
</style>

