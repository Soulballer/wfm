<template>
  <div class="numbers-item">

    <or-checkbox
      v-model="number.checked"

      :class="{'unabled': number.usedData.length}"
      :disabled="readonly || Boolean(number.usedData.length)"
      :key="number.value"
      transition="expand"
    >
      <div class="item-content">
        <div class="item-data">
          <span class="item-value">{{number.value}}</span>
          <!-- can rework logic. show span with label and hide span and show input when we click on edit button. and move input inside span. and we will can delete widht calculator -->
          <span ref="span" class="number-disabled" v-show="inputDisabled">{{number.name}}</span>
          <input
            class="input-element"
            type="text"
            v-model.trim="number.name"
            ref="name"
            v-show="!inputDisabled"
            @blur="updateName"
            :style="{width: getWidth + 'px'}"
          />
        </div>

        <data-numbers
          :data="number"
          :readonly="readonly"
        ></data-numbers>

        <div
          v-if="!number.usedData.length && !readonly"

          class="item-buttons"
        >
          <or-icon-button
            @click="editNumberItem"
            type="flat"
            icon="edit"
          ></or-icon-button>
          <or-icon-button
            @click="removeNumberItem"
            icon="close"
            type="flat"
          ></or-icon-button>
        </div>

        <or-confirm
          :close-on-confirm="!isAllowed"
          :loading="removeProgress"
          @confirm="handleRemove"
          @deny="isAllowed = true; showWarn = false"
          confirmButtonText="Remove"
          ref="confirmRemove"
          title="Remove"
        >
          <or-alert 
            v-show="!isAllowed"
            
            @dismiss="isAllowed = true" 
            type="warning"
          >
            Only Admin has permissions to manage account numbers. Please contact your admin or OneReach Support.
          </or-alert>
          <or-alert 
            v-show="showWarn"

            @dismiss="showWarn = false" 
            type="warning"
          >
            The number cannot be removed as there is a flow activated on it.
          </or-alert>
            Remove {{number.value}} from the global list?
        </or-confirm>

      </div>
    </or-checkbox>

  </div>
</template>

<script>
  import eventHub from '../helpers/eventHub.js';

  import DataNumbers from './dataNumbers.vue';

  export default {
    props: {
      currentFlowDeployedData: {
        type: Object,
        default() {
          return {}
        }
      },
      number: {
        type: Object,
        default() {
          return {}
        }
      },
      readonly: {
        type: Boolean,
        default() {
          return false
        }
      }
    },
    components : { DataNumbers },

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
    methods: {
      getWidth() {
        return this.$refs.span.offsetWidth
      },
      editNumberItem () {
        this.inputDisabled = false;
        this.$nextTick(() => this.$refs.name.focus());
        console.log('aaaaa', this.getWidth(), this.$refs);
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
    }
  }
</script>

<style lang="scss" scoped>
  .numbers-item {
    .ui-checkbox {
      .ui-checkbox__label-text {
        margin-top: 0;
        width: 100%;

        color: black;
    }

    }
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
  }
</style>

