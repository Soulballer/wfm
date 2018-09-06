<template>
  <div class="group" :class="{'unabled' : group.usedData.length}">
    <div class="group__inner-wrapper">
      <or-checkbox
        v-model="group.isSelected"

        :disabled="readonly || Boolean(group.usedData.length)"
        transition="expand"
      ></or-checkbox>

      <or-collapsible
        :class="{'error-class-same-name': invalid}"
        :removeIcon="true"
        @click.stop
        @close="changeOpenState"
        @open="changeOpenState"
      >
        <div slot="header" class="collapsible-header">
            <span ref="span" class="number-disabled" v-show="inputDisabled">{{group.name}}</span>
            <input
              v-show="!inputDisabled"

              :disabled="inputDisabled"
              :value="group.name"
              @blur="updateName"
              @input="checkName"
              class="input-element"
              ref="name"
              type="text"
            />
            <span class="visible ui-icon header-icon material-icons {currentIcon}">{{currentIcon}}</span>
          
          <div v-if="!readonly && !group.usedData.length" class="group-buttons">
            <or-icon-button
              :tooltip="!group.editable ? 'Group is in use, not allowed to edit' : ''"
              @click.stop="editGroup"
              icon="edit"
              type="flat"
            ></or-icon-button>
            <or-icon-button
              :tooltip="!group.editable ? 'Group is in use, not allowed to edit' : ''"
              @click.stop="addToGroup"
              icon="add"
              type="flat"
            ></or-icon-button>
            <or-icon-button
              :tooltip="!group.editable ? 'Group is in use, not allowed to edit' : ''"
              @click.stop="openUngroupConfirm"
              type="flat"
            >
              <svg class="ungroup_button" width="16" height="16" viewBox="0 0 16 16" xmlns="http://www.w3.org/2000/svg">
                <path d="M0 0h2.53v.84h6.73V0h2.53v2.53h-.85V5.9h2.52v-.85H16v2.53h-.84v5.9H16V16h-2.53v-.84h-5.9V16H5.06v-2.53h.84v-2.52H2.52v.84H0V9.25h.84V2.53H0V0zm13.47 7.58v-.84h-2.52v2.52h.84v2.53H9.25v-.85H6.74v2.52h.84v.85h5.9v-.85h.84v-5.9h-.85zm-4.2-5.05v-.85H2.52v.85h-.85v6.73h.85v.85H5.9V7.6h-.85V5.05h2.53v.84h2.53V2.52h-.84zm-1.7 5.05h-.83v2.53h2.52v-.84h.85V6.74H7.6v.84zM.85 1.68h.84V.84H.84v.84zm9.27 0h.85V.84h-.84v.84zM5.9 6.74h.84V5.9H5.9v.84zm8.42 0h.84V5.9h-.84v.84zM5.9 15.16h.84v-.84H5.9v.84zm8.42 0h.84v-.84h-.84v.84zM.84 10.96h.84v-.85H.84v.85zm9.27 0h.85v-.85h-.84v.85z" fill-rule="nonzero" fill="currentColor"/>
              </svg>
            </or-icon-button>
          </div>
        </div>
        <item-content
          v-for="number in group.numbers"

          :group="group"
          :key="number.id"
          :number="number"
          :readonly="readonly"
        ></item-content>
      </or-collapsible>

      <data-numbers
        :data="group"
        :readonly="readonly"
      ></data-numbers>
    </div>

    <or-confirm
      :contain-focus="false"
      @confirm="handleUngroup"
      class="ungroup-modal"
      confirmButtonText="Save"
      ref="confirmUngroup"
      title="Ungroup"
    >
      Ungroup {{group.name}} (move {{group.numbers.length}} phone numbers to the global list)?
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
          class="select-all-button"
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

  </div>  
</template>

<script>
  import eventHub from '../helpers/eventHub.js';

  import ItemContent from './itemContent.vue';
  import DataNumbers from './dataNumbers.vue';

  export default {
    props: {
      allFilteredNumbers: {
        type: Array,
        default: []
      },
      invalid: {
        type: Boolean,
        default: false
      },
      group: {
        type: Object,
        default: {}
      },
      readonly: {
        type : Boolean,
        default: false
      }
    },
    components: { DataNumbers, ItemContent },

    created () {
      eventHub.$on('remove number from group', this.handleNumberRemove)
      eventHub.$on('add numbers to group', this.clearSelected);
      eventHub.$on('open modal add to group', this.handleNumbersList);
    },
    destroyed () {
      eventHub.$off('remove number from group', this.handleNumberRemove);
      eventHub.$off('add numbers to group', this.clearSelected);
      eventHub.$off('open modal add to group', this.handleNumbersList);
    },

    computed: {
      allNumbersSelected() {
        return _.every(this.copyNumbers, n => n.checked);
      },
      currentIcon() {
        return this.open
                ? 'keyboard_arrow_down'
                : 'keyboard_arrow_right'
      },
      numbersSelectedToAdd() {
        return _.filter(this.copyNumbers, num => num.checked)
      },
      selectAllButtonText () {
        return `${this.allNumbersSelected ? 'Uns' : 'S'}elect all (${this.copyNumbers.length})`
      },
    },
    data() {
      return {
        copyNumbers: [],
        errorClass: false,
        inputDisabled: true,
        open: false
      }
    },
    methods: {
      addToGroup() {
        if (!this.group.editable) return

        this.$refs.confirmAddNumber.open();
        eventHub.$emit('open modal add to group');
      },
      buyNewNumbers() {
        this.$refs.confirmAddNumber.close();
        eventHub.$emit('buy new number')
      },
      clearSelected() {
        _.forEach(this.copyNumbers, n => n.checked = false);
      },
      changeOpenState() {
        this.open = !this.open
      },
      checkName(e) {
        this.errorClass = _.isEmpty(e.target.value.trim()) ? true : false;
      },
      editGroup() {
        if (!this.group.editable) return

        this.inputDisabled = false;
        this.$nextTick(() => this.$refs.name.focus());
      },
      handleAddNumber() {
        const {id} = this.group;

        // set group id to selected numbers
        _.forEach(this.numbersSelectedToAdd, n => _.set(n, 'group', id))

        // add selected numbers to group numbers
        this.group.numbers = _.concat(this.group.numbers, this.numbersSelectedToAdd)

        this.$http.put(
          this.$flow.gatewayUrl('identifiers-group', this.$flow.providersAccountId()),
          {
            channel: 'text',
            group: {
              // array of number IDs
              numbers: _.map(this.group.numbers, `value`)
            }
          },
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`,
            },
            params: {
              identifier: this.group.id,
            }
          }
        )
        .then(() => eventHub.$emit('update numbers data'))
        // remove selected numbers from general number list
        .then(() => eventHub.$emit('add numbers to group', this.numbersSelectedToAdd));
      },
      handleNumbersList() {
        this.copyNumbers = _
          .chain(_.cloneDeep(this.allFilteredNumbers))
          .filter(n => !n.usedData.length)
          .map(n => ({...n, checked: false}) )
          .value()
      },
      handleNumberRemove(number) {
        // remove number from group numbers
        this.group.numbers = _.reject(this.group.numbers, number);

        // remove group id from number object
        _.set(number, 'group', undefined);

        this.$http.put(
          this.$flow.gatewayUrl('identifiers-group',
          this.$flow.providersAccountId()),
          {
            channel: 'text',
            group: {
              // array of number IDs
              numbers: _.map(this.group.numbers, 'value')
            }
          },
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`
            },
            params: {
              identifier: this.group.id
            }
          })
          .then(() => eventHub.$emit('update numbers data'))
          .then(() => eventHub.$emit('put number to general list', number))
          .then(() => {
            if (_.size(this.group.numbers) < 2) {
              this.handleUngroup(this.group);
            }
          });
      },
      handleUngroup() {
        const {numbers} = this.group

        // remove group ID from numbers
        _.forEach(numbers, x => _.set(x, 'group', undefined))

        this.$http.delete(
          this.$flow.gatewayUrl('identifiers-group', this.$flow.providersAccountId()),
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`
            },
            params: {
              group: this.group.id,
              channel: 'text'
            }
          }
        )
        .then(() => eventHub.$emit('update numbers data'))
        .then(() => eventHub.$emit('ungroup', this.group))
      },
      openUngroupConfirm() {
        if (!this.group.editable) return 

        this.$refs.confirmUngroup.open();
      },
      selectAll() {
        const condition = _.every(this.copyNumbers, n => n.checked);

        _.forEach(this.copyNumbers, n => { n.checked = !condition })
      },
      updateName() {
        this.inputDisabled = true;
        
        if (this.errorClass) {
          this.$refs.name.value = this.group.name;
          this.errorClass = false;
          return
        }

        _.set(this.group, 'name', this.$refs.name.value)

        this.$http.put(
          this.$flow.gatewayUrl('identifiers', this.$flow.providersAccountId()),
          {
            identifier: {
              name: this.$refs.name.value
            }
          },
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`
            },
            params: {
              identifier: this.group.id
            }
          }
        )
      }
    }
  }
</script>

<style lang="scss" scoped>
  .group {

    &__inner-wrapper {
      display: flex;

      .ui-checkbox {
        align-self: flex-start;
        margin-top: 3px;
        margin-bottom: 0;
      }

      .or-collapsible {
        padding-left: 8px;

        color: black;
        
        border: 0;

        &.error-class-same-name {
          color: #f95d5d !important;
        }

        &:hover {
          .collapsible-header .group-buttons {
            visibility: visible;
          }
        }

        & > .body-wrapper > .body {
          padding: 5px;
        }

        & > .header {
          min-height: 0 !important;
          padding: 0;

          .header-content {
            color: inherit;
          }

          .header-icon {
            margin: 0;

            color: inherit;
          }
        }

        .collapsible-header {
          display: flex;
          align-items: center;

          .input-element {
            font-size: 16px;
            line-height: 22px;
            color: inherit;
            font-weight: bold;

            border: 1px solid white;
            outline: none;
            background: none;
            
            &:not(:disabled) {
              border: 1px solid #64b2da;
            }
          }

          .group-buttons {
            visibility: hidden;
            display: flex;
            position: relative;
            z-index: 20;

            .ungroup_button {
              margin-top: 4px;
            }

            .ui-icon-button {
              height: 25px !important;
              width: 25px !important;
            }
          }
        }
      }
    }

    &.unabled {
      .group__inner-wrapper {
        .or-collapsible {
          color: #91969d;
        }
      }
    }

    &.unabled .ui-checkbox.is-disabled .ui-checkbox__checkmark {
      display: block;
    }

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
  }
</style>


