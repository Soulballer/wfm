<template>
  <div>
    <main-component 
      :all-checked-numbers.sync="schema.allCheckedNumbers"
      :groups.sync="schema.groups"
      :is-admin="schema.isAdmin"
      :is-keywords="schema.isKeywords"
      :is-new="isNew"
      :keywords="schema.keywords"
      :numbers.sync="schema.numbers" 
      :readonly="readonly"
      :selected-elem-length.sync="schema.selectedElemLength" 
      :selected-numbers.sync="schema.selectedNumbers" 
      :selected-groups.sync="schema.selectedGroups">
    </main-component>
  </div>
</template>

<script>
  import { validators } from '_validators';
  import { mapState } from 'vuex';

  import MainComponent from './components/mainComponent.vue';

  const {required, jsExpressionNonEmptyString, generateValidators} = validators;
  

  export default {
    props      : ['isNew', 'readonly', 'schema', 'step', 'stepId', 'steps', 'template'],
    components : { MainComponent },

    created() {
      this.schema.isNew = this.isNew;
      this.schema.isAdmin = this.isAdmin;
    },

    computed: {
      ...mapState({
        isAdmin : ({auth}) => auth.role == 'ADMIN' ? true : false
      })
    },
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
      isNew              : template.isNew,
      keywords           : template.keywords,
      selectedElemLength : template.selectedElemLength
  })};

  export const validator = (template) => {
    return {
      selectedElemLength    : generateValidators(true, {
        custom(data, ctx) {
          if (!ctx.isNew) return Boolean(data) 
           
          return true 
        }
      })
    };
  };

  export const meta = {
    name    : 'test-external-component',
    type    : 'onereach-studio-form-editor',
    version : '2.1.4'
  };
</script>

<style scoped lang="scss">

</style>
