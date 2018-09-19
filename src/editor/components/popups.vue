<template>
  <or-confirm
    :contain-focus="false"
    @confirm="handleRemove"
    confirmButtonText="Move"
    ref="confirmRemove"
    title="Move"
  >
    Move <b>{{number.value}}</b> from <b>{{group.name}}</b> to the global list?
  </or-confirm>

</template>

<script>
  import eventHub from '../helpers/eventHub.js';

  export default {
    created() {
      eventHub.$on('popup open', this.openDelete);
    },

    computed: {
      group() {
        let group  = eventHub.store.deleteNumberModal ? eventHub.store.deleteNumberModal.group : '';
        return group
      },
      number() {
        let number  = eventHub.store.deleteNumberModal ? eventHub.store.deleteNumberModal.number : '';
        return number
      }
    },
    data() {
      return {
      }
    },
    methods: {
      handleRemove() {
        eventHub.$emit(`remove number from group/${this.group.id}`, this.number);
      },
      openDelete() {
        console.log('eventHub', eventHub.store)
        this.$refs.confirmRemove.open();
      }
    }
  } 
</script>

<style lang="scss" scoped>

</style>

