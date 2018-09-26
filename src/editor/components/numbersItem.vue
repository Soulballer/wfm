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
          <span v-show="inputDisabled" class="number-disabled" >{{localNumber}}</span>
          <input
            v-model="localNumber"
            v-show="!inputDisabled"

            @blur="updateName"
            @input="checkName"
            @keyup.13="updateName"
            class="input-element"
            ref="name"
            type="text"
          />  
        </div>
        
        <or-progress-circular 
          v-show="isLoading"
          
          color="primary" 
          size="18" 
          type="indeterminate"
        ></or-progress-circular>

        <data-numbers
          :data="number"
          :readonly="readonly"
        ></data-numbers>

        <div
          v-if="!number.usedData.length && !readonly && isAdmin"

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

  </div>
</template>

<script>
  import eventHub from '../helpers/eventHub.js';

  import DataNumbers from './dataNumbers.vue';

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
    data() {
      return {
        errorClass: false,
        inputDisabled: true
      }
    },
    methods: {
      checkName(e) {
        this.errorClass = _.isEmpty(e.target.value.trim()) ? true : false;
      },
      editNumberItem() {
        this.inputDisabled = false;
        this.$nextTick(() => this.$refs.name.focus());
      },
      removeNumberItem() {
        eventHub.$set(eventHub.store, 'numberToRemove', { ...this.number });
        eventHub.$emit('remove number from account');
      },
      updateName() {
        this.inputDisabled = true;
        
        if (this.errorClass) {
          this.$refs.name.value = this.number.name;
          this.errorClass = false;
          return
        }

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

