<template>
  <div class="group" :class="{'unabled' : group.usedData.length}">
<div class="group__inner-wrapper">
  <or-checkbox
    :disabled="readonly"
    :key="group.name"
    transition="expand"
    v-model="group.isSelected"
    @change="toggleGroupSelection"
    >
     </or-checkbox>
      <or-collapsible
        @open="changeOpenState"
        @close="changeOpenState"
        :removeIcon="true"
        @click.stop>
        <div slot="header" class="collapsible-header">
          <div>
            <input
              class="input-element"
              type="text"
              :value="group.name"
              ref="name"
              :disabled="inputDisabled"
              :size="group.name.length * 0.88"
              @blur="updateName"
              @input="checkName"
              :class="{'error-class-same-name': invalid}"
            ></input>
            <span class="visible ui-icon header-icon material-icons {currentIcon}">{{currentIcon}}</span>
          </div>
          <div class="group-buttons">
            <or-icon-button
              v-if="!readonly && !group.usedData.length"
              class="edit-button"
              type="flat"
              icon="edit"
              :tooltip="group.editable ? '' : 'Group is in use, not allowed to edit'"
              @click="editGroup"
              ></or-icon-button>
            <or-icon-button
              v-if="!readonly && !group.usedData.length"
              class="edit-button"
              type="flat"
              icon="add"
              :tooltip="group.editable ? '' : 'Group is in use, not allowed to edit'"
              @click="addToGroup"
              ></or-icon-button>
            <or-icon-button
              v-if="!readonly && !group.usedData.length"
              class="edit-button"
              type="flat"
              :tooltip="group.editable ? '' : 'Group is in use, not allowed to edit'"
              @click="openUngroupConfirm"
            >
              <svg width="16" height="16" viewBox="0 0 16 16" xmlns="http://www.w3.org/2000/svg">
                <path d="M0 0h2.53v.84h6.73V0h2.53v2.53h-.85V5.9h2.52v-.85H16v2.53h-.84v5.9H16V16h-2.53v-.84h-5.9V16H5.06v-2.53h.84v-2.52H2.52v.84H0V9.25h.84V2.53H0V0zm13.47 7.58v-.84h-2.52v2.52h.84v2.53H9.25v-.85H6.74v2.52h.84v.85h5.9v-.85h.84v-5.9h-.85zm-4.2-5.05v-.85H2.52v.85h-.85v6.73h.85v.85H5.9V7.6h-.85V5.05h2.53v.84h2.53V2.52h-.84zm-1.7 5.05h-.83v2.53h2.52v-.84h.85V6.74H7.6v.84zM.85 1.68h.84V.84H.84v.84zm9.27 0h.85V.84h-.84v.84zM5.9 6.74h.84V5.9H5.9v.84zm8.42 0h.84V5.9h-.84v.84zM5.9 15.16h.84v-.84H5.9v.84zm8.42 0h.84v-.84h-.84v.84zM.84 10.96h.84v-.85H.84v.85zm9.27 0h.85v-.85h-.84v.85z" fill-rule="nonzero" fill="currentColor"/>
              </svg>
            </or-icon-button>
          </div>
        </div>
        <item-content
          v-for="number in group.numbers"
          key="number.value"
          :number="number"
          :group="group"
          :readonly="readonly"
        ></item-content>
      </or-collapsible>
      <group-data-number
        :group="group"
        :readonly="readonly"
      ></group-data-number>
      </div>
  <or-confirm
    title="Ungroup"
    ref="confirmUngroup"
    confirmButtonText="Save"
    @confirm="handleUngroup"
    :contain-focus="false"
  >
    Ungroup {{group.name}} (move {{group.numbers.length}} phone numbers to the global list)?
  </or-confirm>
  <or-confirm
    title="Add numbers"
    ref="confirmAddNumber"
    confirmButtonText="Add"
    @confirm="handleAddNumber"
    :contain-focus="false"
  >
    <or-checkbox
      @change="selectAll"
      v-if="numbers.length > 0"
      class="select-all-button"
      :value="allNumbersSelected"
    >{{selectAllButtonText}}
    </or-checkbox>
    <or-checkbox
      v-for="number in allNumbersUnselected"
      :key="number.value"
      :value="number.checked"
      @change="handleCheckboxChange(number)"
    >
        <span class="item-value">{{number.value}}</span>
        <span class="item-value">{{number.name}}</span>
      </or-checkbox>
  </or-confirm>
</div>  
</template>

<script>
import eventHub from './eventHub.js';

import GroupDataNumber from './groupDataNumber.vue';
import ItemContent from './itemContent.vue';

export default {
  props: {
    allFilteredNumbers: {
      type: Array
    },
    readonly: {
      type : Boolean
    },
    isData: {
      type : Boolean
    },
    group: {
      type: Object
    },
    invalid: {
      type: Boolean,
      default: false
    },
    numbers: {
      type: Array
    }
  },
  components: {GroupDataNumber, ItemContent},

  created () {
    eventHub.$on('remove number from group', this.handleNumberRemove)
    eventHub.$on('add numbers to group', this.clearSelected);
    this.$on('open modal add to group', this.handleNumbersList);
  },
  destroyed () {
    eventHub.$off('remove number from group', this.handleNumberRemove);
    this.$off('open modal add to group', this.handleNumbersList);
  },
  computed: {
    allNumbersSelected () {
      return _.every(this.copyNumbers, n => n.checked);
    },
    allNumbersUnselected () {
      return _.filter(this.copyNumbers, n => !n.usedData.length);
    },
    currentIcon () {
      return this.open
              ? 'keyboard_arrow_down'
              : 'keyboard_arrow_right'
    },
    getWidthOfInput() {
      return this.group.name.length * 10;
    },
    numbersSelectedToAdd () {
      return this.numbers.filter(num => num.checked = _.get(_.find(this.allNumbersUnselected, {value: num.value}), 'checked', false))
    },
    selectAllButtonText () {
      return `${this.allNumbersSelected ? 'Uns' : 'S'}elect all (${this.allNumbersUnselected.length})`
    },
  },
  data () {
    return {
      copyNumbers: [],
      errorClass: false,
      open: false,
      inputDisabled: true
    }
  },
  methods: {
    addToGroup (e) {
      if (this.group.editable) {
        e.stopPropagation();
        this.$refs.confirmAddNumber.open();
        this.$emit('open modal add to group');
      }
    },
    clearSelected () {
      _.forEach(this.allNumbersUnselected, n => n.checked = false);
    },
    changeOpenState () {
      this.open = !this.open
    },
    checkName(e) {
      this.errorClass = false;
      if(_.isEmpty(e.target.value.trim())) {
        this.errorClass = true;
      }
    },
    editGroup (e) {
      if (this.group.editable) {
        if (e) e.stopPropagation();
        this.inputDisabled = false;
        this.$nextTick(() => this.$refs.name.focus());
      }
    },
    handleAddNumber () {
      const {id} = this.group;

      // set group id to selected numbers
      _.forEach(this.numbersSelectedToAdd, n => _.set(n, 'group', id))

      // add selected numbers to group numbers
      this.group.numbers = this.group.numbers.concat(this.numbersSelectedToAdd)

      this.$http.put(
        this.$flow.gatewayUrl('identifiers-group',
        this.$flow.providersAccountId()),
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
    handleCheckboxChange (number) {
      number.checked = !number.checked;
    },
    handleNumbersList () {
      this.copyNumbers = _.forEach(_.cloneDeep(this.allFilteredNumbers), n => {n.checked = false});
    },
    handleNumberRemove (number) {
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
    handleUngroup () {
      const {numbers} = this.group

      // remove group ID from numbers
      _.forEach(numbers, x => _.set(x, 'group', undefined))

      this.$http.delete(
        this.$flow.gatewayUrl('identifiers-group',
        this.$flow.providersAccountId()),
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
    openUngroupConfirm (e) {
      if (this.group.editable) {
        e.stopPropagation(); /*can use modifier .stop*/
        this.$refs.confirmUngroup.open();
      }
    },
    selectAll () {
      const condition = _.every(this.copyNumbers, n => n.checked);
      _.forEach(this.copyNumbers, n => {
        n.checked = !condition;
      })
    },
    toggleGroupSelection (event) {

      if (this.group.usedData.length) {
        this.group.isSelected = false;
        
        return;
      }

      this.group.isSelected = event;
      
      if (this.group.isSelected) {
        _.forEach(this.group.numbers, number => number.checked = true);
      } else {
        _.forEach(this.group.numbers, number => number.checked = false);
      }
    },
    updateName () {
      this.inputDisabled = true;
      this.open = false;
      
      if (this.errorClass) {
        this.$refs.name.value = this.group.name;
        return
      }

      _.set(this.group, 'name', this.$refs.name.value)

      this.$http.put(
        this.$flow.gatewayUrl('identifiers',
        this.$flow.providersAccountId()),
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
        })
    }
  },
}
</script>

<style lang="scss" scoped>
  .group {
    position: relative;
    
    margin-bottom: 2px;
    
    &:not(.unabled) {
      .or-collapsible {
        color: black;
        
        .body-wrapper {
          color: black;
        }
      }
    }
    
    &__inner-wrapper {
      display: flex;
    }
    
    .ui-confirm__message {
      display: flex;
      flex-direction: column;
      
      .ui-checkbox__checkmark {
        display: inline-block;
        
        vertical-align: top;
      }
      
      .ui-checkbox__label-text {
        display: inline-block;
        width: auto;
        
        vertical-align: middle;
      }
    }
  }


.group {
  position: relative;
  
  margin-bottom: 2px;
  
  &:not(.unabled) {
    .or-collapsible {
      color: black;
      
      .body-wrapper {
        color: black;
      }
    }
  }
  
  &__inner-wrapper {
    display: flex;
  }
  
  .ui-confirm__message {
    display: flex;
    flex-direction: column;
    
    .ui-checkbox__checkmark {
      display: inline-block;
      
      vertical-align: top;
    }
    
    .ui-checkbox__label-text {
      display: inline-block;
      width: auto;
      
      vertical-align: middle;
    }
  }

  .ui-modal__body {
    min-height: 0;
  }

  .ui-textbox__input {
    border: none;
    font-size: 16px;
    color: inherit;
    font-weight: bold;
  }
  
  .ui-checkbox {
    display: inline-block;
    
    vertical-align: top;
    
    .ui-checkbox__label-text {
      color: black;
    }
  }

  .ui-checkbox__label-text {
    margin-top: 0 !important;
  }
}
</style>


