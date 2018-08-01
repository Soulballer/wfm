<template>
  <div class="data-numbers">
    <a ref="trigger3" v-show="data.usedData.length && !readonly" class="data-numbers--link">
      <span class="slash">  -  </span>
      <span v-show="!data.anyKeywords"> in use by </span>
      <span v-for="(key, index) in data.keywordsСollision" :key="key" v-if="index < 2">
        <span v-show="!data.anyKeywords">#</span>{{key}}
        <span v-if="index !== 1 && data.keywordsСollision.length !== 1">, </span>
      </span> 
      <span v-if="data.keywordsСollision.length > 2">and {{data.keywordsСollision.length - 2}} other</span>
    </a>

    <or-popover open-on="click" trigger="trigger3" class="data-numbers--popover">
      <p v-for="d in data.usedData" :key="d.botId" class="data-numbers--popover__container">
        <a :href="'/#/bots/' + d.botId" target="_blank">{{d.botName}}</a> /
        <a :href="'/#/flows/' +d.botId + '/' + d.flowId" target="_blank">{{d.flowName}}</a>
      </p>
    </or-popover>
  </div>
</template>

<script>
  export default {
    props: {
      data: {
        type: Object,
        default() {
          return {}
        }
      },
      readonly: {
        type : Boolean,
        default() {
          return false
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  .data-numbers {
    margin-right: 5px;

    color: #91969d;

    .slash {
      font-size: 16px;
      white-space: pre;
    }

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
        margin: 0;
        padding: 10px; 
      }
    } 
  }
</style>

