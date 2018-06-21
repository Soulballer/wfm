<template>
  <div class="main">
    <h3>Flow number(s)</h3>
    <div class="no-numbers" v-if="!anyNumbersAvailable && !localGroups.length">
      <p>There are no numbers in your account.</p> <!--maybe have sense addn CSS class-->
      <div>
        To add a number to your account, please <span :class="{'read-only': readonly}" class="add-new-num" @click="addNewNumber">add new number</span> or 
        <a :class="{'read-only': readonly}" href="mailto:support@onereach.ai">contact support</a>.
      </div>
    </div>
    <div v-else>
      
      <div 
        
        @keydown.enter.prevent="openDropdown"
      >
        <div class="ui-select__display" @click="toggleDropdown" :class="{'error-class-same-name': !localSelectedNumbers.length && !localSelectedGroups.length}">
          <div
            class="ui-select__display-value"
            :class="{ 'is-placeholder': !hasDisplayText }"
          >
            <div>
              <button class="ui-select__display-selected" v-for="text in displayText.split(',')" v-if="text.length">
                {{text}}
                <i @click.stop="!readonly ? removeSelection(text) : ''" class="material-icons ui-select__display-close-button">close</i>
              </button>
            </div>
          </div>
    
          <ui-icon class="ui-select__dropdown-button">
            <i class="material-icons">{{!showDropdown ? 'arrow_drop_down' : 'arrow_drop_up'}}</i>
          </ui-icon>
        </div>
        
        <transition name="ui-select--transition-fade">
          <div
            class="ui-select__dropdown"
            ref="dropdown"
            tabindex="-1"
            
            v-show="showDropdown"
          >
        
            <div class="container" :class="{ 'error-empty-num': !selectedElemLength }">
          
              <div class="search-box" v-show="!readonly">
                <or-textbox :disabled="readonly" class="search-filter" placeholder="Type to search" name="searchInput" v-model="searchValue" icon="search"></or-textbox> <!--use readonly attr for disabling or-textbox-->
                <ui-icon @click.native="clearSearch" class="clear-search" icon="close" v-if="searchValue"></ui-icon>
              </div>
              <ui-progress-linear
                color="primary"
                size="24"
                v-show="isData"
                style="margin-bottom: 14px;"
              ></ui-progress-linear> <!--maybe have sense addn CSS class-->
              <div class="data" v-show="!isData">
                <transition name="expand-filter">
                  <div class="buttons">
                    <div
                      class="button"
                      v-show="hasResultsAfterFilter && showFilterButton"
                      @click="changeFilter"
                    >{{filterButtonText}}
                    </div>
                    <div
                      :class="{'read-only': readonly}"
                      class="button"
                      v-show="localSelectedNumbers.length > 1 && !readonly"
                      @click="createGroup"
                    >Create group
                    </div>
                  </div>
                </transition>
                <div class="numbers-list scrollbar">
                  <div class="ui-select__empty" v-if="!allFilteredNumbers.length && !allFilteredGroups.length && !isLoading && !readonly">
                    <p>No matching phone numbers</p>
                  </div>
                  <div v-else>
                    <or-checkbox
                      key="selectAllNumbers"
                      :disabled="readonly || !(availableNumbers.length + groups.length)"
                      @change="selectAll"
                      v-if="availableNumbers.length || availableGroups.length"
                      v-show="!readonly"
                      class="select-all-button"
                      :class="{ 'all-selected' : allNumbersSelected, 'some-selected' : someFilterNumberChecked}"
                      :value="allNumbersSelected">{{selectAllButtonText}}
                    </or-checkbox>
                    <groups
                      :groups="allFilteredGroups"
                      :readonly="readonly"
                      :isData="isData"
                      :numbers="availableNumbers"
                      :allFilteredNumbers="allFilteredNumbers"
                    ></groups>
                    <numbers-items
                      :currentFlowDeployedData="currentFlowDeployedData"
                      :numbers="allFilteredNumbers"
                      :readonly="readonly"
                      :isData="isData"
                    ></numbers-items>
                  </div>
                </div>
              </div>
              <div
                class="button"
                @click="addNewNumber"
                :class="{'read-only': readonly}"
                v-show="!readonly"
              >+ Add new number</div> <!--please use button and add disabled attribute for readonly mode-->
            </div>
          </div>
        </transition>
            
      </div>
      
    </div>
    <div class="container container_for-modal">
      <buy-modal
        ref="buyModal"
        :numbers="numbers"
        :groups="groups"
        :readonly="readonly"
        ></buy-modal>
    </div>
  </div>
</template>
<script>
    import * as _ from 'lodash';
    import {validators} from '_validators';
    import email from './email.vue';
    import password from './password.vue';

    const {required, jsExpressionNonEmptyString, generateValidators} = validators;

    export default {
        name       : 'editor-test-example',
        props      : ['template', 'schema', 'step', 'stepId', 'steps', 'readonly'],
        components : {email, password},
        computed   : {},

        created () {},

        data () {
            return {};
        },

        methods : {},

        validations () {
            return {
                schema : validator(this.template)
            };
        }

    };

    export const data = (template) => ({
        email               : '',
        emailPlaceholder    : template.emailPlaceholder,
        emailLabel          : template.emailLabel,
        password            : '',
        passwordPlaceholder : template.passwordPlaceholder,
        passwordLabel       : template.passwordLabel,
    });

    export const validator = (template) => {
        return {
            email    : generateValidators(template.validateRequired, {required}),
            password : generateValidators(template.validateRequired, {required}),
        };
    };

    export const meta = {
        name    : 'test-external-component',
        type    : 'onereach-studio-form-editor',
        version : '1.0'
    };
</script>

<style scoped lang="scss" rel="stylesheet/scss">
    @import '../scss/colors.scss';

</style>


<style lang="scss" rel="stylesheet/scss">
    @import '../scss/colors.scss';

</style>
