<template>
  <div class="main">
    <h3>Flow number(s)</h3>

    <div class="no-numbers" v-if="!anyNumbersAvailable && !localGroups.length">
      <p>There are no numbers in your account.</p>
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
              <button class="ui-select__display-selected" v-for="text in displayText.split(',')" :key="text" v-if="text.length">
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
                <or-textbox :disabled="readonly" class="search-filter" placeholder="Search" name="searchInput" v-model="searchValue" icon="search"></or-textbox> <!--use readonly attr for disabling or-textbox-->
                <ui-icon @click.native="clearSearch" class="clear-search" icon="close" v-if="searchValue"></ui-icon>
                <or-icon-button
                  :class="{'create-group__button--inactive': !availableToGroup}"
                  :tooltip="createGroupWarn" 
                  tooltipPosition="left top"
                  @click="createGroup"
                  class="button create-group__button"
                >
                  Create group
                </or-icon-button>
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
                    >
                      {{filterButtonText}}
                    </div>
                  </div>
                </transition>

                <div class="numbers-list scrollbar">
                  <div class="ui-select__empty" v-if="!allFilteredNumbers.length && !allFilteredGroups.length && !readonly">
                    <p>No matching numbers or groups found</p>
                  </div>

                  <div v-else>
                    <or-checkbox
                      :disabled="readonly || !(availableNumbers.length + groups.length)"
                      @change="selectAll"
                      v-if="availableNumbers.length || availableGroups.length"
                      v-show="!readonly"
                      class="select-all-button"
                      :class="{ 'all-selected' : allNumbersSelected, 'some-selected' : someFilterNumberChecked}"
                      :value="allNumbersSelected"
                    >{{selectAllButtonText}}
                    </or-checkbox>

                    <groups
                      :allFilteredNumbers="allFilteredNumbers"
                      :groups="allFilteredGroups"
                      :readonly="readonly"
                    ></groups>

                    <numbers-items
                      :currentFlowDeployedData="currentFlowDeployedData"
                      :numbers="allFilteredNumbers"
                      :readonly="readonly"
                    ></numbers-items>

                    <div class="no-active-numbers" v-show="_.isEmpty(selectedNumbers) && _.isEmpty(selectedGroups) && readonly">
                        You have purchased numbers, but none of them was selected before saving the flow.
                    </div>
                  </div>
                </div>

              </div>
              <div
                class="button"
                @click="addNewNumber"
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
        :numbers="allAvailableNumbers"
      ></buy-modal>
    </div>

  </div>
</template>

<script>
  import Promise from 'bluebird';
  import uuid from 'uuid';
  import eventHub from '../helpers/eventHub.js';

  import BuyModal from './buyModal.vue';
  import Groups from './groups.vue';
  import NumbersItems from './numbersItems.vue';

  

export default {
  name       : 'editor-Wait-for-message',
  components : { BuyModal, Groups, NumbersItems },

  created () {
    eventHub.$on('ungroup', this.handleUngroup);
    eventHub.$on('add numbers to group', this.handleAddNumbersToGroup);
    eventHub.$on('put number to general list', this.putNumberToGeneralList);
    eventHub.$on('remove number from general list', this.removeNumberFromGeneralList);
    eventHub.$on('update numbers data', this.updateNumbersData);
    eventHub.$on('buy new number', this.addNewNumber);
    
  },
  destroyed () {
    eventHub.$off('ungroup', this.handleUngroup);
    eventHub.$off('add numbers to group', this.handleAddNumbersToGroup);
    eventHub.$off('put number to general list', this.putNumberToGeneralList);
    eventHub.$off('remove number from general list', this.removeNumberFromGeneralList);
    eventHub.$off('update numbers data', this.updateNumbersData);
    eventHub.$off('buy new number', this.addNewNumber);
    document.removeEventListener('click', this.onExternalClick);
  },


  computed: {
    availableToGroup() {
      let usedNumber = [];
      
      if (this.localSelectedNumbers.length < 2) {
        this.createGroupWarn = 'Min two numbers needed';
        return false;
      } else if(!_.isEmpty(this.localSelectedNumbers.filter(num => !num.editable))) {
        this.createGroupWarn = 'Adding numbers in use is not allowed';
        return false;
      } else {
        this.createGroupWarn = '';
        return true;
      }
    },
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
      allAvailableNumbers: [],
      anyNumbersAvilable: true,
      createGroupWarn: '',
      currentFlowDeployedData: {},
      currentShowState: this.readonly ? false : this.showAll,
      flowsList: [],
      isData: false,
      isLoading: false,
      localGroups: this.groups,
      localSelectedGroups: this.selectedGroups,
      localNumbers: _.clone(this.numbers),
      localSelectedNumbers: this.selectedNumbers,
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
            for (let stepNumber in step.flowNumberKeywords) {
              if (stepNumber == number.value && !step.isKeywords) {
                number.usedData.push(step);
                number.checked = false;
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
            for (let stepNumber in step.flowNumberKeywords) {
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
            for (let stepNumber in step.flowNumberKeywords) {
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
                  number.checked = false;
                }
              }
            }
          })
        });
        
        this.localGroups.map(group => {
          this.flowsList.forEach(step => {
            for (let stepNumber in step.flowNumberKeywords) {
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
      if (!this.availableToGroup) return 
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
      this.localSelectedNumbers = _.reject(this.localNumbers, x => x.checked === true);

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
      //if (!this.localNumbers.length) this.isLoading = true;
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
          
          this.allAvailableNumbers = responseJson
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
                  checked: _.get(_.find(self.selectedNumbers, {
                    id: number.id
                  }), 'checked', false),
                  keywordsСollision: [],
                  group: number.group,
                  editable: true
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

          const usedAnywhereGroup = [], usedAnywhereNumber = [];
          this.flowsList.filter(flow => _.forIn(flow.flowNumberKeywords, (val, key) => key.indexOf('+') < 0 ? usedAnywhereGroup.push(key) : usedAnywhereNumber.push(key)));
          
          _.forEach(this.localGroups, g => {g.editable = !usedAnywhereGroup.some(id => id === g.id)});
          _.forEach(this.localNumbers, n => {n.editable = !usedAnywhereNumber.some(id => id === n.value)});

          this.keywordsСollisionData();
          this.isData = false;
      });
    },
    updateSelectedElemLength () {
      let selectedLength = this.selectedNumbers.length + this.selectedGroups.length;
      //this.schema.selectedElemLength = this.selectedElemLength;
      this.$emit('update:selectedElemLength', selectedLength);
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
    isKeywords: {
      type: Boolean
    },
    keywords: {
      type: Array
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
    template: {
      type: Object
    },
    schema: {
      type: Object
    },
    step: {
      type: Object
    },
    steps: {
      type: Array
    },
    stepId: {
      type: String
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
      //this.schema.allCheckedNumbers = this.allLocalCheckedNumbers;
      this.allCheckedNumbers = this.allLocalCheckedNumbers;
      this.$emit('update:allCheckedNumbers', this.allLocalCheckedNumbers);
    },
    localNumbers: {
      handler() {
        let selectedNumbers = _.filter(this.localNumbers, n => n.checked);
        this.localSelectedNumbers = selectedNumbers;
        
        if( JSON.stringify(selectedNumbers) !==  JSON.stringify(this.selectedNumbers) ) {
          //this.schema.selectedNumbers = selectedNumbers;
          this.selectedNumbers = selectedNumbers;
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
        //this.schema.selectedGroups = selectedGroups;
        //this.selectedGroups = selectedGroups;
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
      //console.log('keywords added', this.isKeywords, this.schema.isKeywords)
        this.updataKeywordsСollisionData();
      },
      deep: true,
    },
    isKeywords: {
      handler() {
        //console.log('keyword handler')
        this.updataKeywordsСollisionData();
      }
    }
  },


};

</script>

<style scoped lang="scss" rel="stylesheet/scss">
.ui-modal__container {
    width: 55rem;
  }
  .ui-modal__body {
    display: flex;
  }
  .main {
    margin-bottom: 30px;
  }

  h3 {
    margin-bottom: 15px;
    font-size: 16px;
  }

  .no-numbers {
    margin-bottom: 40px;
    color: #5E656D;
    font-size: 16px;
    line-height: 23px;
    
    p {
      color: #0F232E;
    }
    a, .add-new-num {
      color: #65b3da;
      white-space: nowrap;
    }
    .add-new-num {
      cursor: pointer;
    }
  }

  .read-only {
    pointer-events: none;
    cursor: default;
    color: rgba(0,0,0,.38) !important;
  } 

  .data {
    padding: 0 10px;

    & ~ .button {
      padding-top: 10px !important;
    }
  }

  .container{
    padding:2px 0 20px;
    border-radius:4px;

    &_for-modal {
      border: none;
      padding: 0;
      margin: 0;
    }
  }

  .ui-select {
  &__display {
    border: 1px solid #dfdfdf;
    border-radius: 3px;
    background-color: #f6f6f6;
    min-height: 36px;
    align-items: center;
    color: #0f232e;
    cursor: pointer;
    display: flex;
    font-weight: normal;
    padding: 2px 10px;
    transition: border 0.1s ease;
    user-select: none;
    width: 100%;
    transition: background-color 0.5s ease-out;
    
    .is-placeholder {
      position: relative;
      
      &:after {
        content: "Select numbers (groups)";
        
        position: absolute;
        top: -10px;
        
        font-size: 14px;
      }
    }
    
    &:hover {
      background-color: white;
    }
  }
  
  &__dropdown {
    position: relative;
    
    width: auto;
    
    border: 1px solid #dfdfdf;
    border-top: 0;
    box-shadow: 0 3px 16px 0 rgba(0, 0, 0, 0.08);
    
    z-index: 10;
    
    &-button {
      font-size: 2.125rem;
      
      i {
        font-size: 35px;
      }
    }
  }
  
  &__display-selected {
    position: relative;

    margin: 2px 4px 2px 0;
    padding: 5px 20px 5px 4px;

    border: none;
  }

  &__display-close-button {
    position: absolute;
    top: 7px;
    right: 2px;

    font-size: 15px;
    color: #8C9492;

    cursor: pointer;
  }
}
.ui-select__empty {
  display: flex;
  justify-content: center;
  margin: 40px 0;

  p {
    display: inline-block;
    margin-bottom: 0;

    color: rgba(94, 101, 109, 0.55);
    font-size: 14px;

    vertical-align: center;
  }
}

.container{
  padding:2px 0 20px;
  border-radius:4px;
    &_for-modal {
      border: none;
      padding: 0;
      margin: 0;
    }
  .search-box {
    position:relative;
    .clear-search {
      position: absolute;
      right: 95px;
      top: 17%;
      z-index: 2;
      cursor: pointer;
      font-size: 20px;
      color:#a7aaaf;
    }
    .ui-textbox {
      position:relative;

      margin-bottom: 5px;
      
      &__input {
        padding-left:25px;
        padding-right:125px;
        padding-top: 7px;
        font-size:14px;
        border: none;
        background-color: white;
      }
      &__icon {
        &-wrapper {
          .ui-icon.search {
            color: #0594ed !important;
            font-size: 18px;
            line-height: 14px;
          }
          position:absolute;
          top: 6px;
          left: 5px;

          margin: 0;
          padding: 0;
          z-index: 10;
        }
      }
    }

    .create-group__button {
      position: absolute;
      top: 8px;
      right: 6px;

      width: auto;
      height: auto;

      font: inherit;  
      font-size: 12px;

      background-color: white;
      border-radius: 0;
      cursor: pointer;

      &:hover {
        background-color: white;
      }

      .ui-icon-button__icon {
        font: inherit;
        color: inherit
      }

      .ui-ripple-ink {
        display: none;
      }

      &--inactive {
        color: #91969d;

        cursor: default;
      }
    }
  }
  .button {
    padding: 0 10px;
    color: #0594ed;
    box-sizing: content-box;
    font-size: 12px;
    cursor: pointer;

    &.disabled {
      cursor:no-drop;
      color:#e7e7e7;
    }
  }

  .line {
    width:calc(100% + 24px);
    margin-left:-12px;
    border-bottom:1px solid #e7e7e7;
  }
  .ui-checkbox {
    align-items:center;
    
    &__label {
      &-text {
        margin-top:5px;
        white-space:nowrap;
        overflow:hidden;
        text-overflow:ellipsis;
      }
    }
  }
  .ui-alert {
    white-space: normal;
  }
  .numbers-list {
    font-size:14px;
    min-height: 30px;
    max-height: 180px;
    overflow-y:auto;
    overflow-x:hidden;
    margin:7px 0 0;
  }
  .ui-checkbox {
    margin: 5px 0;

    &__label {
      &-text {
        font-size:14px;
        line-height:normal;
      }
    }
    &.is-disabled {
      .ui-checkbox__checkmark {
        display:none;
      }
    }
    &:not(.is-disabled):not(.checked) {
      .ui-checkbox {
        &__checkmark {
          &:before {
            background-color: #f7f7f7 !important;
            border: 1px solid #c7c7c7;
          }
          &:after {
            border-right-color: #132530;
            border-bottom-color: #132530;
          }
        }
      }
      &.active,
      &:hover {
        .ui-checkbox {
          &__checkmark {
            &:before {
              border-color: rgba(0,0,0,.54);
            }
          }
        }
      }
      &.checked {
        .ui-checkbox {
          &__checked {
            &:after {
              border-right-color: #132530;
              border-bottom-color: #132530;
            }
          }
        }
      }
    }
    &.select-all-button {
      font-weight:bold;
      height: 25px;
      
      .ui-checkbox__label-text {
        color: black;
      }

      &:not(.all-selected) {
        &.some-selected {
          .ui-checkbox {
            &__checkmark {
              &:after {
              -webkit-transform: rotate(90deg);
              opacity:1;
              transform: rotate(90deg);
              border-right:2px solid;
              border-bottom: none;
              height: 11px;
              bottom: 6px;
              }
            }
          }
        }
      }
    }
  }
  .scrollbar {
    &::-webkit-scrollbar {
        width: 6px;
        background:#f6f6f6;
    }
    &::-webkit-scrollbar-track {
        -webkit-border-radius: 10px;
        border-radius: 10px;
    }
    &::-webkit-scrollbar-thumb {
        -webkit-border-radius: 4px;
        border-radius: 4;
        background:#d8d8d8;
    }
    &::-webkit-scrollbar-thumb:window-inactive {
    	background: #d8d8d8;
    }
  }

  .no-active-numbers {
    margin-top: 10px;

    font-size: 15px;
  }
}

</style>
