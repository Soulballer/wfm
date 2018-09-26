<template>
  <div>
    <!-- INSIDE GROUP POPUS -->
    <or-confirm
      :contain-focus="false"
      @confirm="handleRemove"
      confirmButtonText="Move"
      ref="confirmRemove"
      title="Move"
    >
      Move <b>{{number.value}}</b> from <b>{{group.name}}</b> to the global list?
    </or-confirm>

    <!-- GROUP POPUPS -->
    <or-confirm
      :contain-focus="false"
      @confirm="handleUngroup"
      confirmButtonText="Save"
      ref="confirmUngroup"
      title="Split"
    >
      Split <b>{{splitGroupData.name}}</b> (move {{splitGroupData.numbers.length}} phone numbers to the global list)?
    </or-confirm>

    <or-confirm
      :contain-focus="false"
      @confirm="handleAddNumber"
      class="addnumber-modal"
      confirmButtonText="Add"
      ref="confirmAddNumber"
      title="Add numbers"
    >
      <div v-if="copyNumbers.length">
        <or-checkbox
          :value="allNumbersSelected"
          @change="selectAll"
        >
          {{selectAllButtonText}}
        </or-checkbox>
        <or-checkbox
          v-for="number in copyNumbers"
          v-model="number.checked"

          :key="number.id"
        >
          <span class="item-value">{{number.value}} {{number.name}}</span>
        </or-checkbox>
      </div>
      <div v-else>
        All your available numbers are already in the groups.<br>
        You can <a @click="buyNewNumbers" class="button">buy new numbers</a> or split one of your existing groups.
      </div>
    </or-confirm>

    <or-confirm
      :close-on-confirm="removeProgress"
      :loading="removeProgress"
      @confirm="handleRemoveNumber"
      class="remove-modal"
      confirmButtonText="Yes, release"
      denyButtonText="Keep number"
      ref="confirmRemove"
      title=""
      type="success"
    >
      <h4>Remove {{numberToRemove.value}} </h4>

      <p>You may have flows subscribed to this number.</p>
      <p>If the number is released, your flows will still be active but will not respond to this number.</p>
      <p>Are you sure you want to release <b>{{numberToRemove.value}}</b>?</p>
    </or-confirm>

  </div>
</template>

<script>
  import eventHub from '../helpers/eventHub.js';

  export default {
    props: {
      allFilteredNumbers: {
        type: Array,
        default: []
      }
    },
    created() {
      eventHub.$on('add number to group', this.addNumberToGroup);
      eventHub.$on('remove number from account', this.removeNumber);
      eventHub.$on('remove number from group popup', this.removeNumberFromGroup);
      eventHub.$on('split group', this.splitGroup);
    },
    destroyed () {
      eventHub.$off('add number to group', this.addNumberToGroup)
      eventHub.$off('remove number from account', this.removeNumber);
      eventHub.$off('remove number from group popup', this.removeNumberFromGroup);
      eventHub.$off('split group', this.splitGroup);
    },

    computed: {
      addNumber() {
        return eventHub.store.deleteNumberFromGroup ? eventHub.store.deleteNumberFromGroup.group : '';
      },
      addNumbersData() {
        return eventHub.store.addNumbersData ? eventHub.store.addNumbersData : '';
      },
      allNumbersSelected() {
        return _.every(this.copyNumbers, n => n.checked);
      },
      group() {
        return eventHub.store.deleteNumberFromGroup ? eventHub.store.deleteNumberFromGroup.group : '';
      },
      number() {
        return eventHub.store.deleteNumberFromGroup ? eventHub.store.deleteNumberFromGroup.number : '';
      },
      numbersSelectedToAdd() {
        return _.filter(this.copyNumbers, num => num.checked)
      },
      numberToRemove() {
       return eventHub.store.numberToRemove ? eventHub.store.numberToRemove : {value : ''};
      },
      selectAllButtonText () {
        return `${this.allNumbersSelected ? 'Uns' : 'S'}elect all (${this.copyNumbers.length})`
      },
      splitGroupData() {
        return eventHub.store.splitGroupData ? eventHub.store.splitGroupData : {numbers : ''};
      }
    },
    data() {
      return {
        copyNumbers: [],
        removeProgress: false
      }
    },
    methods: {
      addNumberToGroup() {
        this.handleNumbersList();
        this.$refs.confirmAddNumber.open();
      },
      buyNewNumbers() {
        this.$refs.confirmAddNumber.close();
        eventHub.$emit('buy new number')
      },
      clearSelected() {
        _.forEach(this.copyNumbers, n => n.checked = false);
      },
      handleAddNumber() {
        const {id} = this.addNumbersData.group;

        // set group id to selected numbers
        _.forEach(this.numbersSelectedToAdd, n => _.set(n, 'group', id))

        // add selected numbers to group numbers
        this.addNumbersData.group.numbers = _.concat(this.addNumbersData.group.numbers, this.numbersSelectedToAdd)

        this.$http.put(
          this.$flow.gatewayUrl('identifiers-group', this.$flow.providersAccountId()),
          {
            channel: 'text',
            group: {
              // array of number IDs
              numbers: _.map(this.addNumbersData.group.numbers, `value`)
            }
          },
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`,
            },
            params: {
              identifier: this.addNumbersData.group.id,
            }
          }
        )
        // remove selected numbers from general number list
        .then(() => this.clearSelected())
        .then(() => eventHub.$emit('update numbers data'));
      },
      handleNumbersList() {
        this.copyNumbers = _
          .chain(_.cloneDeep(this.addNumbersData))
          .filter(n => n.editable && n.value)
          .map(n => ({...n, checked: false}))
          .value()
      },
      handleRemove() {
        eventHub.$emit(`remove number from group/${this.group.id}`, this.number);
      },
      handleRemoveNumber() {
        this.removeProgress = true;

        this.$http.delete(  
          this.$flow.gatewayUrl('identifiers',
          this.$flow.providersAccountId()),
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`
            },
            params: {
              identifier: this.numberToRemove.value
            }
          }
        )
        .then(() => eventHub.$emit('update numbers data'))
        .then(() => {
          this.removeProgress = false;
        })
        .catch(() => {
          this.removeProgress = false;
        });
      },
      handleUngroup() {
        const { numbers } = this.splitGroupData

        // remove group ID from numbers
        _.forEach(numbers, x => _.set(x, 'group', undefined))

        this.$http.delete(
          this.$flow.gatewayUrl('identifiers-group', this.$flow.providersAccountId()),
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`
            },
            params: {
              group: this.splitGroupData.id,
              channel: 'text'
            }
          }
        )
        .then(() => eventHub.$emit('update numbers data'))
      },
      removeNumber() {
        this.$refs.confirmRemove.open();
      },
      removeNumberFromGroup() {
        this.$refs.confirmRemove.open();
      },
      selectAll() {
        const condition = _.every(this.copyNumbers, n => n.checked);

        _.forEach(this.copyNumbers, n => { n.checked = !condition })
      },
      splitGroup() {
        this.$refs.confirmUngroup.open();
      }
    }
  } 
</script>

<style lang="scss" scoped>
  .remove-modal {
    .ui-modal {
      &__header {
        display: none;
      }

      &__body {
        padding: 0 !important;

        h4 {
          margin-bottom: 45px;
          padding: 12px 22px;

          color: white;
          
          background-color:#F35958;
        }
        
        p {
          padding: 0 22px;

          &:last-child {
            margin-bottom: 130px;
          }
        }
      }
    

      &__footer {
        button.ui-button.ui-button--type-secondary.ui-button--color-primary {
          color: white;

          border-color: #F35958;
          background-color: #F35958;

          &:hover {
            border-color: #A35958;
            background-color: #A35958;
          }
        }
      }
    }
  }

  .addnumber-modal {
    .item-value {
      color: black;
      font-weight: normal;
    }

    .ui-checkbox {
      .ui-checkbox__label-text {
        color: black;
        font-weight: normal;
      }

      .ui-checkbox {
        &__checkmark {
          &:before {
            background-color: #f7f7f7 !important;
            border: 1px solid #c7c7c7;
          }
          &:after {
            border-color: #132530 !important;
            border-right-color: #132530;
            border-bottom-color: #132530;
          }
        }
      }
    }

    .ui-modal .ui-modal__container > .ui-modal__body {
      padding: 16px 24px;

      .button {
        padding: 0;
        
        font-size: 15px;

        &:hover {
          color: #0594ed;
        }
      }
    }
  }
</style>

