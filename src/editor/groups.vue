<template>
  <div class="groups">
  <group-item
    v-for="(group, index) in groups"
    :group="group"
    :numbers="numbers"
    :allFilteredNumbers="allFilteredNumbers"
    key="group.name"
    :invalid="localGroupsNameCollision[index] && groups.length > 1"
    :readonly="readonly"
  ></group-item>
</div>  
</template>

<script>
import GroupItem from './groupItem.vue';

export default {
  components: {GroupItem},

  data() {
    return {
      localGroupsNameCollision: [],
    }
  },
  methods: {
    checkCollision() {
      let invalidFlag = false;
      _.forEach(this.groups, (stap1, index1) => {
        Vue.set(this.localGroupsNameCollision, index1, false);
        let invalidFlag = false;
        _.forEach(this.groups, (stap2, index2) => {
          if (index1 !== index2 && stap1.name === stap2.name) {
            invalidFlag = true;
          }
        });
          if (invalidFlag) {
            Vue.set(this.localGroupsNameCollision, index1, true);
          }
      });
    }
  },
  mounted() {
    this.checkCollision();
  },
  watch: {
    groups: {
      handler() {
        this.checkCollision();
      },
      deep: true
    }
  },
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
    groups: {
      type: Array
    },
    numbers: {
      type: Array
    }
  }
}
</script>

<style lang="scss" scoped>
  .groups {
    display: flex;
    flex-direction: column;

     .ui-checkbox {
      align-items: flex-start !important;
      margin-bottom: 0 !important;
      
      &.is-disabled {
        .ui-checkbox__checkmark {
          display:none;
        }
      }
    }
  }

  .groups {
  display: flex;
  flex-direction: column;
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
