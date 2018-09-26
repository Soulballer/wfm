<template>
  <div class="item-content">
    <span class="item-value">{{number.value}}</span>
    <span v-show="inputDisabled" class="item-name" >{{localNumber}}</span>
    <div>
      <input
        v-model="localNumber"
        v-show="!inputDisabled"

        :disabled="inputDisabled"
        @blur="updateName"
        @input="checkName"
        @keyup.13="updateName"
        class="input-element"
        ref="name"
        type="text"
      />
    </div>

    <div v-if="!readonly && !group.usedData.length && isAdmin" class="item-buttons">
      <or-icon-button
        :tooltip="!group.editable ? 'Group is in use, not allowed to edit' : ''"
        @click="editNumberItem"
        icon="edit"
        type="flat"
      ></or-icon-button>
      <or-icon-button
        :tooltip="!group.editable ? 'Group is in use, not allowed to edit' : ''"
        @click="removeNumberItem"
        icon="delete_forever"
        type="flat"
      ></or-icon-button>
    </div>
  </div>
</template>

<script>
  import eventHub from '../helpers/eventHub.js';

  export default {
    props: {
      isAdmin: {
        type: Boolean,
        default() {
          return false
        }
      },
      group: {
        type: Object
      },
      number: {
        type: Object
      },
      readonly: {
        type : Boolean
      }
    },
    
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
        if (this.group.editable) {
          this.inputDisabled = false;
          this.$nextTick(() => this.$refs.name.focus());
        }
      },
      handleRemove() {
        eventHub.$emit(`remove number from group/${this.group.id}`, this.number);
      },
      removeNumberItem() {
        eventHub.$set(eventHub.store, 'deleteNumberFromGroup', {number: this.number, group: this.group});
        eventHub.$emit('remove number from group popup');
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
  .item-content {
    position: relative;

    height: 25px;
    display: flex;
    align-items: center;
    padding-right: 55px;
    margin-right: 15px;

    &:hover .item-buttons {
      visibility: visible;
    }

    .input-element {
      padding: 0;
      margin-right: 55px;

      font-family: inherit;
      font-size: 14px;
      color: inherit;
      
      border: none;
      outline: none;
      
      &:not(:disabled) {
        border: 1px solid #64b2da;
      }
    }

    .item-value {
      margin-right: 0.5em;
    }

    .item-name {
      text-overflow: ellipsis;
      white-space: nowrap;
      overflow: hidden;
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
