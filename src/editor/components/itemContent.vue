<template>
  <div class="item-content">
    <span class="item-value">{{number.value}}</span>
    <span v-show="inputDisabled" class="number-disabled" >{{localNumber}}</span>
    <div>
      <input
        v-model="localNumber"
        v-show="!inputDisabled"

        :disabled="inputDisabled"
        @blur="updateName"
        @input="checkName"
        class="input-element"
        ref="name"
        type="text"
      />
    </div>

    <div v-if="!readonly && !group.usedData.length" class="item-buttons">
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

    <or-confirm
      :contain-focus="false"
      @confirm="handleRemove"
      confirmButtonText="Move"
      ref="confirmRemove"
      title="Move"
    >
      Move <b>{{number.value}}</b> from <b>{{group.name}}</b> to the global list?
    </or-confirm>
  </div>
</template>

<script>
  import eventHub from '../helpers/eventHub.js';

  export default {
    props: {
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
        //console.log('-----', eventHub.store)
        eventHub.$emit('remove number from group', this.number);
      },
      removeNumberItem() {
        eventHub.$set(eventHub.store, 'deleteNumberModal', {number: this.number, group: this.group});
        if (this.group.editable) this.$refs.confirmRemove.open();
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
