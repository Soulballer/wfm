<template>
  <div class="data-numbers">
    <a ref="flowlist-popover" v-show="number.usedData.length && !readonly" class="data-numbers--link">
      <span class="slash">-</span>
      <span v-show="!number.anyKeywords">
        in use by 
      </span>
      <span v-for="(key, index) in number.keywordsСollision" v-if="index < 2">
        <span v-show="!number.anyKeywords"> # </span>{{key}}
        <span v-if="index !== 1 && number.keywordsСollision.length !== 1">, </span>
      </span> 
      <span v-if="number.keywordsСollision.length > 2">and {{number.keywordsСollision.length - 2}} other</span>
    </a>
    <or-popover open-on="click" trigger="flowlist-popover" class="data-numbers--popover">
      <div class="data-numbers--popover__container">
        <p v-for="num in number.usedData" class="data-numbers--popover__data">
          <a :href="'/#/bots/' + num.botId" target="_blank">{{num.botName}}</a> /
          <a :href="'/#/flows/' +num.botId + '/' + num.flowId" target="_blank">{{num.flowName}}</a>
        </p>
      </div>
    </or-popover>
  </div>
</template>

<script>
export default {
  name: 'DataNumbers',
  props: {
    readonly: {
      type : Boolean
    },
    number: {
      type: Object
    }
  }
}
</script>

<style lang='scss' scoped>
  .data-numbers {
    display: inline-block;

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
        padding: 20px; 
        margin-top: -10px;
      }

      &__data {
        margin: 10px 0 0 0;
      }
    } 
  }
</style>

