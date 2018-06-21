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
  import Promise from 'bluebird';
  import * as _ from 'lodash';
  import {validators} from '_validators';

  const {required, jsExpressionNonEmptyString, generateValidators} = validators;
  const eventHub = new Vue();

export default {
  name       : 'editor-test-example',
  props      : ['template', 'schema', 'step', 'stepId', 'steps', 'readonly'],
  components : {email, password},
  created () {
    eventHub.$on('ungroup', this.handleUngroup);
    eventHub.$on('add numbers to group', this.handleAddNumbersToGroup);
    eventHub.$on('put number to general list', this.putNumberToGeneralList);
    eventHub.$on('remove number from general list', this.removeNumberFromGeneralList);
    eventHub.$on('update numbers data', this.updateNumbersData);
    
  },
  destroyed () {
    eventHub.$off('ungroup', this.handleUngroup);
    eventHub.$off('add numbers to group', this.handleAddNumbersToGroup);
    eventHub.$off('put number to general list', this.putNumberToGeneralList);
    eventHub.$off('remove number from general list', this.removeNumberFromGeneralList);
    eventHub.$off('update numbers data', this.updateNumbersData);
    document.removeEventListener('click', this.onExternalClick);
  },


  computed: {
    allLocalCheckedNumbers() {
      return _.concat(this.selectedNumbers, ...this.selectedGroups.map(group => group.numbers));
    },
    allLocalCheckedNumbersHierarchy() {
      return _.concat(this.selectedNumbers, ...this.selectedGroups);
    },
    anyNumbersAvailable() { 
      return !!this.localNumbers.length;
    },
    allFilteredGroups () {
      return _.filter(this.localGroups, group => 
        this.isAvilableIfNotChacked(group.isSelected) && !_.isEmpty(this.groupAvailableBySearch(group)));
    },
    allFilteredNumbers () {
      return _.filter(this.localNumbers, number =>
        this.isAvilableIfNotChacked(number.checked) && this.avilableBySearch(number));
    },
    allNumbersSelected () {
      return  _.every(this.availableNumbers, n => n.checked) && _.every(this.availableGroups, g => g.isSelected)
    },
    availableNumbers () {
      //let allNumbers = _.concat(this.localNumbers, ...this.localGroups.map(group => group.numbers));
      return _.filter(this.allFilteredNumbers, n => {
        return n.usedData.length === 0;
      });
    },
    availableGroups () {
      return _.filter(this.allFilteredGroups, g => {
        return g.usedData.length === 0;
      });
    },
    displayText () {
      if (this.allLocalCheckedNumbersHierarchy.length > 0) {
        return _.chain(this.allLocalCheckedNumbersHierarchy)
          .map(value => value.name)
          .join(',')
          .value();
      }
    
      return '';
    },
    filteredNumbers () {
      let availableNumbers =  _.concat(this.localNumbers, ...this.groups.map(group => group.numbers));
      return _.filter(availableNumbers, n => 
        { /* can remove {} and return */
          return this.avilableBySearch(n) &&
            (this.currentShowState ? true : n.checked) &&
            !n.usedData.length
        });
    },
    filterButtonText () {
      return `Show  ${this.currentShowState ? this.selectedNumbersToShow : 'all'}`
    },
    groupsSelected () {
      return _.filter(this.groups, x => x.isSelected);
    },
    hasDisplayText() {
      return Boolean(this.displayText.length);
    },
    hasResultsAfterFilter () {
      const avilableBySearch = _.concat(_.filter(this.localNumbers, n => this.avilableBySearch(n)), _.filter(this.localGroups, group => this.groupAvailableBySearch(group)));
      const allInTheSameState = _.every(avilableBySearch, n => n.checked == avilableBySearch[0].checked);
      return !allInTheSameState;
    },
    selectAllButtonText () {
      return `${this.allNumbersSelected && (this.availableNumbers.length + this.availableGroups.length !== 0) ? 'Uns' : 'S'}elect all (${this.availableNumbers.length + this.availableGroups.length})`
    },
    selectedNumbersToShow () {
      return ('selected (' + (parseInt(_.filter(this.localSelectedNumbers, n => this.avilableBySearch(n)).length)
      + parseInt(_.filter(this.localSelectedGroups, n => this.groupAvailableBySearch(n)).length)) + ')');
    },
    showFilterButton () {
      return (!this.readonly && Boolean(this.localSelectedNumbers.length) &&
        (this.localSelectedNumbers.length !== this.localNumbers.length || Boolean(this.localGroups.length))) || 
        (!this.readonly && Boolean(this.localSelectedGroups.length))
    },
    someFilterNumberChecked () {
      return _.some(this.filteredNumbers, n => n.checked);
    }
  },
  data() {
    return {
      anyNumbersAvilable: true,
      currentFlowDeployedData: {},
      currentShowState: this.readonly ? false : this.showAll,
      flowsList: [],
      isData: false,
      isLoading: false,
      localGroups: this.groups,
      localSelectedGroups: this.selectedGroups,
      localNumbers: _.clone(this.numbers),
      localSelectedNumbers: _.cloneDeep(this.selectedNumbers),
      searchValue: '',
      showDropdown: false
    }
  },
  methods: {
    onExternalClick(e) {
      if (!this.$el.contains(e.target)) {
        if (this.showDropdown) {
          this.closeDropdown();
        }
      }
    },
    keywordsСollisionData () {
      if (!this.isKeywords) {
        this.localNumbers.map(number => {
          this.flowsList.forEach(step => {
            for (stepNumber in step.flowNumberKeywords) {
              if (stepNumber == number.value && !step.isKeywords) {
                number.usedData.push(step);
                //number.checked = false;
                if (!_.isArray(number['keywordsСollision'])) {
                  number['keywordsСollision'] = []
                }
                number['keywordsСollision'][0] = 'in use by any keyword';
                number['anyKeywords'] = true;
              } 
            }
          });
        });
        
        this.localGroups.map(group => {
          this.flowsList.forEach(step => {
            for (stepNumber in step.flowNumberKeywords) {
              if (stepNumber == group.id && !step.isKeywords) {
                if (!group.hasOwnProperty('usedData')) {
                  group.usedData = []
                }
                group.usedData.push(step);
                group.isSelected = false;
                if (!_.isArray(group['keywordsСollision'])) {
                  group['keywordsСollision'] = []
                }
                group['keywordsСollision'][0] = 'in use by any keyword';
                group['anyKeywords'] = true;
              } 
            }
          });
        });
        
      } else {
        this.localNumbers.map(number => {
          this.flowsList.forEach(step => {
            for (stepNumber in step.flowNumberKeywords) {
              if (stepNumber == number.value) {
                let flagKeyword = false;
                this.keywords.forEach((keywordItem) => {
                  let keyword = keywordItem.keyword.slice(1, -1);
                  if (step.flowNumberKeywords[stepNumber].indexOf(keyword) !== -1) {
                    if (!_.isObject(step['flowNumberKeywordsСollision'])) {
                      step['flowNumberKeywordsСollision'] = {}
                    }
                    if (!_.isArray(step['flowNumberKeywordsСollision'][stepNumber])) {
                      step['flowNumberKeywordsСollision'][stepNumber] = [];
                    }
                    if (!_.isArray(number['keywordsСollision'])) {
                      number['keywordsСollision'] = []
                    }
                    if (number['keywordsСollision'].indexOf(keyword) === -1) {
                      number['keywordsСollision'].push(keyword);
                    }
                    step['flowNumberKeywordsСollision'][stepNumber].push(keyword);
                    flagKeyword = true;
                  }
                });
                if (flagKeyword) {
                  number.usedData.push(step);
                  //number.checked = false;
                }
              }
            }
          })
        });
        
        this.localGroups.map(group => {
          this.flowsList.forEach(step => {
            for (stepNumber in step.flowNumberKeywords) {
              if (stepNumber == group.id) {
                let flagKeyword = false;
                this.keywords.forEach((keywordItem) => {
                  let keyword = keywordItem.keyword.slice(1, -1);
                  if (step.flowNumberKeywords[stepNumber].indexOf(keyword) !== -1) {
                    if (!_.isObject(step['flowNumberKeywordsСollision'])) {
                      step['flowNumberKeywordsСollision'] = {}
                    }
                    if (!_.isArray(step['flowNumberKeywordsСollision'][stepNumber])) {
                      step['flowNumberKeywordsСollision'][stepNumber] = [];
                    }
                    if (!_.isArray(group['keywordsСollision'])) {
                      group['keywordsСollision'] = []
                    }
                    if (group['keywordsСollision'].indexOf(keyword) === -1) {
                      group['keywordsСollision'].push(keyword);
                    }
                    step['flowNumberKeywordsСollision'][stepNumber].push(keyword);
                    flagKeyword = true;
                  }
                });
                if (flagKeyword) {
                  group.usedData.push(step);
                  group.isSelected = false;
                }
              }
            }
          })
        });
      }
    },
    updataKeywordsСollisionData () {
      this.localNumbers.map(number => {
        number.usedData = [];
        number['keywordsСollision'] = []
      });
      
      this.localGroups.map(group => {
        group.usedData = [];
        group['keywordsСollision'] = []
      });
      
      this.keywordsСollisionData();
    },
    addNewNumber () {
      this.$refs.buyModal.openModal();
    },
    avilableBySearch (n) {
      if (!n || !n.value) return;
      const parts = n.value.match(/\+?(\w+)/gi);

      const number = parts.shift();
      const query = this.searchValue.toLowerCase();
      
       if (n.name.indexOf(query) > -1 || _.some(n.name.split(' '), a => a.toLowerCase().indexOf(query) === 0)){
        return n;
      }
      
      return number.indexOf(query) > -1 ||
      _.some(parts, a => a.toLowerCase().indexOf(query) === 0) ||
      parts.join(' ').toLowerCase().indexOf(query) === 0;
    },
    groupAvailableBySearch (g) {
      if (!g || !g.name) return;
      
      const query = this.searchValue.toLowerCase();
      if (g.name.indexOf(query) > -1 || _.some(g.name.split(' '), a => a.toLowerCase().indexOf(query) === 0)){
        return g;
      }
      
      const groupNum = _.concat([], ...g.numbers);
      
      return groupNum.filter(number => this.avilableBySearch(number));
    },
    changeFilter () {
      this.currentShowState = !this.currentShowState;
    },
    clearSearch () {
      this.searchValue = "";
    },
    createGroup () {
      const groupId = uuid.v4();

      // assign 'group' prop for every selected number
      _.forEach(this.selectedNumbers, n => _.set(n, 'group', groupId));
      
      let nextNum = _.map(this.localGroups, ({name}) => 
        parseInt(parseInt(name.split("").reverse().join("")).toString().split("").reverse().join("")));
        nextNum = _.isEmpty(nextNum) ? 1 : Math.max(...nextNum) + 1;
        nextNum = _.isNaN(nextNum) ? 1 : nextNum;
      
      // add selectedNumbers to a new group
      // this.localGroups.push({
      //   name: `New group ${this.groups.length}`,
      //   id: groupId,
      //   isSelected: false,
      //   numbers: this.selectedNumbers
      // });
      
      this.$http.post(
        this.$flow.gatewayUrl('identifiers-group',
        this.$flow.providersAccountId()),
        {
          channel: 'text',
          group: {
            id: groupId,
            name: `New group ${nextNum}`,
            // array of number IDs
            numbers: _.map(this.selectedNumbers, x => x.value)
          }
        },
        {
          headers: {
            Authorization: `USER ${this.$settings.token}`
          }
        })
        .then(() => eventHub.$emit('update numbers data'))

      // remove selected numbers from numbers list
      this.localNumbers = _.reject(this.localNumbers, x => x.checked === true);

      // uncheck selected numbers
      _.forEach(this.localSelectedNumbers, x => {
        x.checked = false;
      });
    },
    isAvilableIfNotChacked (condition) {
      return this.currentShowState ? true : condition
    },
    handleAddNumbersToGroup (numbers) {
      // remove numbers added to group from general list
      _.forEach(numbers, n => this.localNumbers = _.reject(this.localNumbers, n))
    },
    handleUngroup (group) {
      // put group numbers back to general number list
      this.localNumbers = _.sortBy(this.localNumbers.concat(group.numbers), n => n.value)
      // remove group from groups list
      this.localGroups = _.reject(this.localGroups, group)
    },
    numbersTotalBy (f) {
      if (!f) return this.localNumbers.length;
      return _.filter(this.localNumbers, f).length;
    },
    putNumberToGeneralList (number) {
      // put number back to general list from group list
      this.localNumbers = _.sortBy(this.localNumbers.concat(number), n => n.value)
    },
    removeNumberFromGeneralList (number) {
      this.localNumbers = _.reject(this.localNumbers, number);
    },
    removeSelection (optionName) {
      this.localSelectedNumbers = _.forEach(this.localSelectedNumbers, number => {
        if (number.name === optionName) {
          number.checked = false;
        }
      });
      
      this.localSelectedGroups = _.forEach(this.localSelectedGroups, group => {
        if (group.name === optionName) {
          group.isSelected = false;
        }
      });
    },
    selectAll () {
      if (this.readonly) return;
      const condition = _.every(this.availableNumbers, n => n.checked) && _.every(this.availableGroups, g => g.isSelected);

      // numbers
      _.forEach(this.availableNumbers, n => {
        n.checked = !condition;
      });

      // groups
      _.forEach(this.availableGroups, n => {
        n.isSelected = !condition;
      });

      //_.forEach(this.localGroups, group => _.forEach(group.numbers, number => number.checked = !condition));
      
      if (!this.numbersTotalBy(n => n.checked) && !this.currentShowState && !this.readonly) this.currentShowState = true;
    },
    toggleDropdown() {
      this[this.showDropdown ? 'closeDropdown' : 'openDropdown']();
    },
    openDropdown() {
      if (this.disabled) {
          return;
      }

      this.showDropdown = true;
    },
    closeDropdown() {
      this.showDropdown = false;
    },
    updateNumbersData () {
      //this.isData = true;
      if (!this.localNumbers.length) this.isLoading = true;
      let self = this;

      this.$http.get(this.$flow.gatewayUrl('identifiers', this.$flow.providersAccountId()), {
          headers: {
            Authorization: `USER ${this.$settings.token}`
          },
          params: {
            groupDetails: 'true',
            channel: 'text'
          }
        })
        .then(response => response.json())
        .then(function(responseJson) { /*can use arrow function*/

          const groupsData =
             _.filter(responseJson, x => x.isGroup)

          const numbersData =
            _.reject(responseJson, x => x.isGroup)

          const numbers =
            _.chain(numbersData)
              .map(function(number) { /*can use arrow function*/
                return {
                  id: number.id,
                  name: number.name,
                  value: number.phoneNumber,
                  usedData: [],
                  checked: _.get(_.find(self.numbers, {
                    id: number.id
                  }), 'checked', false),
                  keywordsСollision: [],
                  group: number.group
                };
              })
              .sortBy('value')
              .value();

          const numbersReducer =
            x => x.group !== '' && x.group !== undefined;

          const numbersInGroups =
            _.filter(numbers, numbersReducer)

          const numbersOutGroups =
            _.reject(numbers, numbersReducer)

          const groups =
            _.map(groupsData, g => ({
              name: g.name,
              id: g.id,
              usedData: [],
              editable: true,
              keywordsСollision: [],
              isSelected: _.get(_.find(self.selectedGroups, {
                    id: g.id
                  }), 'isSelected', false),
              numbers: _.filter(numbersInGroups, n => n.group === g.id)
            }))

          if (JSON.stringify(groups) !== JSON.stringify(this.groups)) {
            this.$emit('update:groups', groups);
            self.localGroups = groups; // assign groups and numbers
          }
          if (JSON.stringify(numbersOutGroups) !== JSON.stringify(this.numbers)) {
            this.$emit('update:numbers', numbersOutGroups);
             self.localNumbers = numbersOutGroups; // assign groups and numbers
          }
    
          this.isData = false;
        });

      const flows = this.$flow.api.flows;
      const deployments = this.$flow.api.deployments;
      const bots = this.$flow.api.bots;
      const currentFlowId = this.$flow.id;

      if(!this.flowsList.length) this.isData = true;

      return Promise.props({
        flows : flows.getFlows({"projection": ['id','data.label']}),
        deployments: deployments.getDeployments({"projection": ['botId', 'flowId', 'data.triggers']}),
        bots: bots.getBots({"projection": ['id','data.label']})
      }).then(result => {
        this.flowsList = _.chain(result.deployments)
          .filter(deployment => {
            if (deployment.flowId === currentFlowId) {
              this.currentFlowDeployedData = deployment;
              return false;
            }

            return _.find(deployment.data.triggers, trigger => {
              if (trigger.params.hasOwnProperty('name')
                  && trigger.params.name.match(/^in\/text\//g)) return true;
            });
          })
          .map(deployment => {
            const flowNumbers = deployment.data.triggers.map((trigger) => trigger.params.name.split("/")[2]);
            const flowKeywords = deployment.data.triggers.map((trigger) => trigger.params.name.split("/")[3]);
            const isKeywords = flowKeywords[0] === undefined ? false : true;
            
            const flowNumberKeywords = flowNumbers.reduce((flowNumberKeywords, currentItem, index) => {
              if(_.isArray(flowNumberKeywords[currentItem])) {
                flowNumberKeywords[currentItem].push(flowKeywords[index]);
              } else {
                flowNumberKeywords[currentItem] = [];
                flowNumberKeywords[currentItem].push(flowKeywords[index]);
              }
              return flowNumberKeywords;
            }, {});
            const botId = _.get(deployment, 'botId', []);
            const flow =_.find(result.flows, {id: deployment.flowId});
            const botName = _.find(result.bots, {id: botId});

            return {
              flowId : flow.id,
              flowName  : _.get(flow, 'data.label', 'No Name'),
              usedNumbers : flowNumbers,
              keywords : flowKeywords,
              flowNumberKeywords : flowNumberKeywords,
              botId: botId,
              botName: botName.data.label,
              isKeywords
            }
          })
          .value();
          const usedAnywhere = [];
          this.flowsList.filter(flow => _.forIn(flow.flowNumberKeywords, (val, key) => key.indexOf('+') < 0 ? usedAnywhere.push(key) : ''));
          _.forEach(this.localGroups, g => {g.editable = !usedAnywhere.some(id => id === g.id)});
          this.keywordsСollisionData();
          this.isData = false;
      });
    },
    updateSelectedElemLength () {
      this.selectedElemLength = this.selectedNumbers.length + this.selectedGroups.length;
      this.$emit('update:selectedElemLength', this.selectedElemLength);
    }
  },
  mounted() {
    this.updateNumbersData();
    document.addEventListener('click', this.onExternalClick);
  },
  props: {
    allCheckedNumbers: {
      type: Array,
      default () {
        return [];
      }
    },
    numbers: {
      type: Array,
      default () {
        return [];
      }
    },
    groups: {
      type: Array,
      default () {
        return [];
      },
    },
    selectedNumbers: {
      type: Array,
      twoWay: true,
      default () {
        return [];
      },
    },
    selectedGroups: {
      type: Array,
      twoWay: true,
      default () {
        return [];
      },
    },
    selectedElemLength: {
      type: Number,
      default: 0
    },
    readonly: {
      type: Boolean,
    },
    showAll: {
      type: Boolean,
      default () {
        return true;
      },
    },
    keywords: {
      type: Array,
      default () {
        return [];
      }
    },
    isKeywords: {
      type: Boolean,
      default () {
        return false;
      }
    },
    v: {
      type: Object,
      default() {
        return {
          schema: {
            selectedElemLength: {},
          }
        }
      }
    }
  },
  watch: {
    allLocalCheckedNumbers() {
      this.$emit('update:allCheckedNumbers', this.allLocalCheckedNumbers);
    },
    localNumbers: {
      handler() {
        let selectedNumbers = _.filter(this.localNumbers, n => n.checked);
        this.localSelectedNumbers = selectedNumbers;
        
        if( JSON.stringify(selectedNumbers) !==  JSON.stringify(this.selectedNumbers) ) {
          this.$emit('update:selectedNumbers', selectedNumbers);
        }
      },
      deep: true,
    },
    selectedNumbers: {
      handler() {
        this.updateSelectedElemLength();
      },
    },
    selectedGroups: {
      handler() {
        this.updateSelectedElemLength();
      },
    },
    localGroups: {
      handler() {
        let selectedGroups = _.filter(this.localGroups, n => n.isSelected);
        this.localSelectedGroups = selectedGroups;
        this.$emit('update:selectedGroups', selectedGroups);
      },
      deep: true,
    },
    // filteredNumbers () {
    //   if (!this.numbersTotalBy(n => n.checked) && !this.currentShowState && !this.readonly) this.currentShowState = true;
    // },
    currentShowState () {
      if (!this.readonly)
        this.$emit('update:showAll', this.currentShowState);
    },
    keywords: {
      handler() {
        this.updataKeywordsСollisionData();
      },
      deep: true,
    },
    isKeywords: {
      handler() {
        this.updataKeywordsСollisionData();
      }
    }
  },

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
