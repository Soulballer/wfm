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
          <span v-show="inputDisabled" ref="span" class="number-disabled" >{{localNumber}}</span>
          <input
            v-model="localNumber"
            v-show="!inputDisabled"

            @blur="updateName"
            class="input-element"
            ref="name"
            type="text"
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
      </div>
    </or-checkbox>
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

    computed: {
      localNumber: {
        get() {
          return this.number.name;
        }
      }
    },
    mounted(){
      console.log('aaaaa-------', this.$refs.handleRemove)
      document.body.appendChild(this.$refs.handleRemove);
    },
    data() {
      return {
        inputWidth: 0,
        inputDisabled: true,
        isAllowed: true,
        isDeactivatingThisFlow: false,
        isDeployed: false,
        removeProgress: false,
        showWarn: false,
        showModal: false
      }
    },
    methods: {
      editNumberItem() {
        this.inputDisabled = false;
        this.$nextTick(() => this.$refs.name.focus());
        this.getWidth();
      },
      getWidth() {
        this.inputWidth = _.get(this.$refs, 'span.offsetWidth', 0);
      },
      handleRemove() {
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
      removeNumberItem() {
        this.$refs.confirmRemove.open();
        this.showModal = true;
      },
      updateName() {
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

    .item-content {
      position: relative;

      display: flex;

      line-height: 24px;

      .item-data {
        font-weight: normal;
      }

      .item-element {
        padding: 0;

        line-height: 22px;

        border: 1px solid #64b2da;
      }

      &:hover .item-buttons {
        visibility: visible;
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

