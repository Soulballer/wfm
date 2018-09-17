<template>
  <div class="groups">
    <group-item
      v-for="(group, index) in groups"

      :allFilteredNumbers="allFilteredNumbers"
      :invalid="localGroupsNameCollision[index]"
      :is-admin="isAdmin"
      :isLoading="isLoading"
      :group="group"
      :key="group.id"
      :readonly="readonly"
    ></group-item>
  </div>  
</template>

<script>
  import GroupItem from './groupItem.vue';

  export default {
    props: {
      allFilteredNumbers: {
        type: Array,
        default() {
          return []
        }
      },
      groups: {
        type: Array,
        default() {
          return []
        }
      },
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
      readonly: {
        type : Boolean,
        default() {
          return false;
        }
      }
    },
    components: { GroupItem },

    data() {
      return {
        localGroupsNameCollision: [],
      }
    },
    methods: {
      checkNameCollision() {
        _.forEach(this.groups, (step1, index1) => {
          this.localGroupsNameCollision.splice(index1, 1, false);

          _.forEach(this.groups, (step2, index2) => {
            if (index1 !== index2 && step1.name === step2.name) {
              this.localGroupsNameCollision.splice(index1, 1, true);
              return false;
            }
          });
        });
      }
    },
    watch: {
      groups: {
        handler() {
          this.checkNameCollision();
        },
        deep: true,
        immediate: true
      }
    }
  }
</script>

<style lang="scss" scoped>

</style>
