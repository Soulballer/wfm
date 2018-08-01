<template>
  <or-checkbox
    :value="isInBuyList"
    @change="updateBuyList"
    class="number-to-buy-item number-container"
  >
    <div class="number-to-buy-item__number">
      <span>{{number.phoneNumber}}</span>
      <!-- <span>$ {{number.price}} monthly</span> -->
    </div>
  </or-checkbox>
</template>

<script>
  import eventHub from '../helpers/eventHub.js';

  export default {
    name: 'NumbersToBuyItem',
    props: {
      buyList: {
        type: Array,
        default: []
      },
      number: {
        type: Object,
        default: {}
      }
    },

    computed: {
      isInBuyList() {
        return _.find(this.buyList, x => x.phoneNumber === this.number.phoneNumber) !== undefined;
      }
    },
    methods: {
      updateBuyList(event) {
        eventHub.$emit('buyList:update',
        event ? _.concat(this.buyList, this.number)
              : _.reject(this.buyList, this.number))
      }
    }
  }
</script>

<style lang='scss' scoped>
  .number-to-buy-item {
    &.number-container {
      &.ui-checkbox {
        margin-bottom: 13px;
      }

      .ui-checkbox {
        &__checkmark {
          width: 19px;
          height: 19px;

          &:after {
            left: 6px;
            top: 1px;

            width: 7px;
            height: 13px;

            border-width: 2px !important;
            border-color: #132530 !important;
          }
        }

        &__label-text {
          margin-left: 13px;
          margin-top: 0;
        }
      }
    }

    &__number {
      span {
        color: #0F232E;
        line-height: 21px;
        font-weight: 400;
      }
    }
  }
</style>

