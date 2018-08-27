<template>
  <div class="item-content">
    <span class="item-value">{{number.value}}</span>
    <div>
      <input
        v-model="number.name"

        :disabled="inputDisabled"
        :size="number.name.length * 0.75"
        @blur="updateName"
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
      Move {{number.value}} from {{group.name}} to the global list?
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

    data() {
      return {
        inputDisabled: true
      }
    },
    methods: {
      editNumberItem() {
        if (this.group.editable) {
          this.inputDisabled = false;
          this.$nextTick(() => this.$refs.name.focus());
        }
      },
      handleRemove() {
        eventHub.$emit('remove number from group', this.number);
      },
      removeNumberItem() {
        if (this.group.editable) this.$refs.confirmRemove.open();
      },
      updateName() {
        this.number.name = this.$refs.name.value;

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
