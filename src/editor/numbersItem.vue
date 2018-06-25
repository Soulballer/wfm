<template>
  <or-checkbox
  @change="numberSelect"
  :disabled="readonly"
  :class="{'unabled': number.usedData.length}"
  :key="number.value"
  transition="expand"
  v-model="number.checked">
  <div class="item-content">
    <div class="item-data">
      <span class="item-value">{{number.value}}</span>
      <!-- can rework logic. show span with label and hide span and show input when we click on edit button. and move input inside span. and we will can delete widht calculator -->
      <input
        class="input-element"
        type="text"
        v-model.trim="number.name"
        ref="name"
        v-show="!inputDisabled"
        @blur="updateName"
        :style="{ width: getWidthOfInput + 'px' }"
      ></input>
      <span class="number-disabled" v-show="inputDisabled">{{number.name}}</span>
    </div>
    <ui-progress-circular
      :size="20"
      color="primary"
      v-show="isData && !readonly"
    ></ui-progress-circular>
    <DataNumbers
      :number="number"
      :readonly="readonly"
    ></DataNumbers>
    <div
      class="item-buttons"
      v-if="!isData && !number.usedData.length">
      <or-icon-button
        v-if="!readonly"
        class="edit-button"
        type="flat"
        icon="edit"
        @click="editNumberItem"
        ></or-icon-button>
        <or-icon-button
          v-if="!readonly"
          class="remove-button"
          type="flat"
          icon="close"
          @click="removeNumberItem"
          ></or-icon-button>
        </div>
        <or-confirm
          title="Remove"
          ref="confirmRemove"
          confirmButtonText="Remove"
          :close-on-confirm="!isAllowed"
          @confirm="handleRemove"
          @deny="isAllowed = true; showWarn = false"
          :loading="removeProgress"
        >
        <or-alert @dismiss="isAllowed = true" type="warning" v-show="!isAllowed">
          Only Admin has permissions to manage account numbers. Please contact your admin or OneReach Support.
        </or-alert>
        <or-alert @dismiss="showWarn = false" type="warning" v-show="showWarn">
          The number cannot be removed as there is a flow activated on it.
        </or-alert>
          Remove {{number.value}} from the global list?
        </or-confirm>
      </div>
    </or-checkbox>
</template>

<script>
import DataNumbers from './DataNumbers';

export default {
  name: 'NumbersItem',
  data() {
    return {
      inputDisabled: true,
      isAllowed: true,
      isDeactivatingThisFlow: false,
      isDeployed: false,
      removeProgress: false,
      showWarn: false
    }
  },
  computed: {
    getWidthOfInput() {
      return this.number.name.length * 10;
    }
  },
  methods: {
    numberSelect() {
      if (this.number.usedData.length) {
        this.number.checked = false;
      }
    },
    editNumberItem () {
      this.inputDisabled = false;
      this.$nextTick(() => this.$refs.name.focus());
    },
    handleRemove () {
      // remove group id
      _.set(this.number, 'group', undefined);
      
      if (!_.isEmpty(this.currentFlowDeployedData)) {
        this.isDeactivatingThisFlow = this.currentFlowDeployedData.data.triggers[0].params.name.split('/')[2] == this.number.value; 
      }

      if (!this.number.usedData.length && !this.isDeactivatingThisFlow) {
        this.removeProgress = true;
        this.$http.delete(
          this.$flow.gatewayUrl('identifiers',
          this.$flow.providersAccountId()),
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`
            },
            params: {
              identifier: this.number.value
            }
          }
        )
        .then(() => eventHub.$emit('update numbers data'))
        .then(() => {
          this.removeProgress = false;
          return eventHub.$emit('remove number from general list', this.number)
        })
        .catch((e) => {
          if (e.status == '403') {
            this.isAllowed = false;
          }
          this.removeProgress = false;
        });
      } else if(this.isDeactivatingThisFlow) {
        this.showWarn = true;
      } else {
        this.isDeployed = true;
      }
    },
    removeNumberItem () {
      this.$refs.confirmRemove.open();
    },
    updateName () {
      _.set(this.number, 'name', this.$refs.name.value);
        
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
            identifier: this.number.value
          }
        })

      this.inputDisabled = true;
    }
  },
  props: {
    currentFlowDeployedData: {
      type: Boolean
    },
    number: {
      type: Object
    },
    isData: {
      type: Boolean
    },
    readonly: {
      type: Boolean
    }
  }
}
</script>

<style lang="scss" scoped>
  .unabled {
    .ui-checkbox__label-text {
      color: #91969d;
    }
      
    .ui-checkbox__checkmark {
      display: inline-block !important;

      &:before {
        background-color: #eee !important;
        border: 1px solid rgba(0,0,0,.38) !important;
        
        opacity: 0.3 !important;
        cursor: default;
      }
    }
    
    .ui-ripple-ink {
      display: none;
    }
  }

  .item-content {
    position: relative;
    height: 25px;

    display: flex;
    align-items: center;

    &:hover .item-buttons {
      visibility: visible;
    }

    .input-element {
      border: none;
      outline: none;
      font-family: inherit;
      font-size: 14px;
      color: inherit;
      padding: 0;
      
      &:not(:disabled) {
        border: 1px solid #64b2da;
      }
    }

    .item-value {
      margin-right: 0.5em;
    }
}

.item-data {
  display: flex;
  justify-content: flex-start;
}

.item-buttons {
  position: absolute;
  right: 0;
  top: 0;
  visibility: hidden;
  background-color: white;

  .ui-icon-button {
    height: 25px !important;
    width: 25px !important;
  }
}
</style>

