<template>
  <div class="item-content">
  <span class="item-value">{{number.value}}</span>
  <div>
    <input
      class="input-element"
      type="text"
      v-model="number.name"
      ref="name"
      :disabled="inputDisabled"
      @blur="updateName"
      :style="{ width: getWidthOfInput + 'px' }"
    ></input>
  </div>
  <ui-progress-circular
    :size="20"
    color="primary"
    v-show="isData && !readonly"
  ></ui-progress-circular>
  <div
    class="item-buttons"
    v-if="!isData && !group.usedData.length">
    <or-icon-button
      v-if="!readonly"
      class="edit-button"
      type="flat"
      icon="edit"
      :tooltip="group.editable ? '' : 'Group is in use, not allowed to edit'"
      @click="editNumberItem"
    ></or-icon-button>
    <or-icon-button
      v-if="!readonly"
      class="remove-button"
      type="flat"
      icon="delete_forever"
      :tooltip="group.editable ? '' : 'Group is in use, not allowed to edit'"
      @click="removeNumberItem"
    ></or-icon-button>
  </div>
  <or-confirm
    title="Move"
    ref="confirmRemove"
    confirmButtonText="Move"
    @confirm="handleRemove"
    :contain-focus="false"
  >Move {{number.value}} from {{group.name}} to the global list?
  </or-confirm>
</div>
</template>

<script>
import eventHub from './helpers/eventHub.js';

export default {
  props: {
    readonly: {
      type : Boolean
    },
    number: {
      type: Object
    },
    isData: {
      type : Boolean
    },
    group: {
      type: Object
    }
  },
  computed: {
    getWidthOfInput() {
      return this.number.name.length * 10;
    }
  },
  data() {
    return {
      inputDisabled: true
    }
  },
  methods: {
    editNumberItem () {
      if (this.group.editable) {
        this.inputDisabled = false;
        this.$nextTick(() => this.$refs.name.focus());
      }
    },
    handleRemove () {
      eventHub.$emit('remove number from group', this.number);
    },
    removeNumberItem () {
      if (this.group.editable) this.$refs.confirmRemove.open();
    },
    updateName () {
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

.buttons {
  display: flex;
  justify-content: space-between;
}
</style>
