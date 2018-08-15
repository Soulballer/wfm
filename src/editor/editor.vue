<template>
  <main-component 
    :all-checked-numbers.sync="schema.allCheckedNumbers"
    :groups.sync="schema.groups"
    :is-keywords="schema.isKeywords"
    :keywords="schema.keywords"
    :numbers="schema.numbers" 
    :readonly="readonly" 
    :selected-numbers.sync="schema.selectedNumbers" 
    :selected-groups.sync="schema.selectedGroups">
  </main-component>
</template>

<script>
  import {validators} from '_validators';

  import MainComponent from './components/mainComponent.vue';

  const {required, jsExpressionNonEmptyString, generateValidators} = validators;
  

  export default {
    name       : 'editor-Wait-for-message',
    props      : ['template', 'schema', 'step', 'stepId', 'steps', 'readonly'],
    components : { MainComponent },
    validations () {
      return {
        schema : validator(this.template)
      };
    }
  }

  export const data = (template) => {
    return({
      allCheckedNumbers  : template.allCheckedNumbers,
      selectedNumbers    : template.selectedNumbers,
      numbers            : template.numbers,
      groups             : template.groups,
      selectedGroups     : template.selectedGroups,
      showAll            : template.showAll,

      keywords           : template.keywords,
      selectedElemLength : template.selectedElemLength
  })};

  export const validator = (template) => {
    return {
      // email    : generateValidators(template.validateRequired, {required}),
      // password : generateValidators(template.validateRequired, {required}),
    };
  };

  export const meta = {
    name    : 'test-external-component',
    type    : 'onereach-studio-form-editor',
    version : '1.0'
  };
</script>

<style scoped lang="scss">

</style>
