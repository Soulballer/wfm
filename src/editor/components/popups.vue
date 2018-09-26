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
          <span>{{number.value}} {{number.name}}</span>
        </or-checkbox>
      </div>
      <div v-else>
        All your available numbers are already in the groups.<br>
        You can <a @click="buyNewNumbers" class="button">buy new numbers</a> or split one of your existing groups.
      </div>
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
      eventHub.$on('remove number from group popup', this.removeNumberFromGroup);
      eventHub.$on('split group', this.splitGroup);
    },
    destroyed () {
      eventHub.$off('add number to group', this.addNumberToGroup)
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
      selectAllButtonText () {
        return `${this.allNumbersSelected ? 'Uns' : 'S'}elect all (${this.copyNumbers.length})`
      },
      splitGroupData() {
        return eventHub.store.splitGroupData ? eventHub.store.splitGroupData : {numbers : ''};
      }
    },
    data() {
      return {
        copyNumbers: []
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
        console.log(this.copyNumbers)
      },
      handleRemove() {
        eventHub.$emit(`remove number from group/${this.group.id}`, this.number);
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
  .addnumber-modal {
    .item-value {
      color: black;
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

