<template>
  <div class="group-numbers">
  <a ref="trigger3" v-show="group.usedData.length && !readonly" class="group-numbers--link">
    <span class="slash"> -</span>
    <span v-show="!group.anyKeywords">
      in use by 
    </span>
    <span v-for="(key, index) in group.keywordsСollision" v-if="index < 2">
      <span v-show="!group.anyKeywords">#</span>{{key}}
      <span v-if="index !== 1 && group.keywordsСollision.length !== 1">, </span>
    </span> 
    <span v-if="group.keywordsСollision.length > 2">and {{group.keywordsСollision.length - 2}} other</span>
  </a>
  <or-popover open-on="click" trigger="trigger3" class="group-numbers--popover">
      <div class="group-numbers--popover__container">
        <p v-for="gr in group.usedData" class="group-numbers--popover__data">
          <a :href="'/#/bots/' + gr.botId" target="_blank">{{gr.botName}}</a> /
          <a :href="'/#/flows/' +gr.botId + '/' + gr.flowId" target="_blank">{{gr.flowName}}</a>
        </p>
      </div>
  </or-popover>
</div>
</template>

<script>
export default {
  props: {
    readonly: {
      type : Boolean
    },
    group: {
      type: Object
    }
  }
}
</script>

<style lang="scss" scoped>
    .group-numbers {
    display: inline-block;
    margin-right: 5px;

    color: #91969d;

    &--link {
      display: inline-block;
      
      font-size: 14px;
      font-weight: normal;
    
      cursor: pointer;
      vertical-align: middle;
      white-space: nowrap;
    
      span {
        display: inline-block;
        
        line-height: normal;
        
        vertical-align: middle;
      }
    }

    &--popover {
      &__container {
        padding: 10px 20px 20px; 
      }

      &__data {
        margin: 10px 0 0 0;
      }
    } 
  }
</style>

