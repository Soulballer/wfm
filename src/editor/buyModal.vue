<template>
  <or-modal
  title="Buy numbers"
  class="buyModal"
  ref="modal"
  :contain-focus="false"
  @close="isAllowed = true"
  >
  <div class="left"></div>
  <div class="right"></div>
  <div class="map">
    <span>Select state on the map: <b>{{selectedState.name}}</b></span>
    <states-map class="map" :available-numbers="numbers" :selected-state.sync="selectedState" ></states-map>
    <div>
      <p>Your active numbers</p>
      <ul class="state-numbers scrollbar">
        <li v-for="number in numbersFilteredByState">{{number.value}} <span>{{number.state}}</span></li>
      </ul>
    </div>
  </div>
  <div class="list">
    <or-alert @dismiss="isAllowed = true" type="warning" v-show="!isAllowed">
      Only Admin has permissions to manage account numbers. Please contact your admin or OneReach Support.
    </or-alert>
    <!-- <or-select
      label="Choose state"
      name="state"
      v-model="selectedStateName"
      :options="statesNames"
      class="statesSelect"
      @change="requestNumbers(selectedState)"
    ></or-select> -->
    <div 
      class="search-box" 
      v-show="!readonly">
      <or-textbox
        :disabled="readonly" 
        class="search-filter" 
        placeholder="Type to search" 
        name="searchInput" 
        v-model="searchValue" 
        icon="search"
      ></or-textbox>
      <ui-icon 
        @click.native="clearSearch" 
        class="clear-search" 
        icon="close" 
        v-if="searchValue"
      ></ui-icon>
    </div>
    <div class="numbers-container">
      <ui-progress-linear
        color="primary"
        size="24"
        v-show="isLoading"
        style="margin-bottom: 14px;"
      ></ui-progress-linear>
      <div v-if="!isLoading">
        <div
          class="button"
          v-show="hasItemsInBuyList"
          @click="changeBuyFilter"
        >{{buyfilterButtonText}}</div>
        <div class="numbers-list scrollbar">
          <div 
            class="ui-select__empty" 
            v-show="!filteredNumbers.length"
          >
            <p>No matching phone numbers</p>
          </div>
          <number-to-buy-item
            v-for="number in filteredNumbers"
            :number="number"
            :buy-list.sync="buyList"
            :key="number.phoneNumber"
            :states="states"
            transition="expand">
          </number-to-buy-item>
        </div>
      </div>
    </div>
    <div class="footer">
      <or-button
        color="primary"
        @click="buyNumbers"
        :disabled="!hasItemsInBuyList || buyProgress"
      >Buy {{hasItemsInBuyList ? buyList.length : null}}
      </or-button>
      <!-- <div class="total">
        Total: USD {{totalPrice}} mothly
      </div> -->
    </div>
    <ui-progress-circular
      color="primary"
      type="indeterminate"
      class="handle-progress"
      v-show="buyProgress"
    ></ui-progress-circular>
  </div>
  </or-modal> 
</template>

<script>
  import * as d3 from '../../node_modules/d3/build/d3.js';
  //import {uStates, clickedState} from './uStates.js';
  import eventHub from './eventHub.js';
  import NumberToBuyItem from './numberToBuyItem.vue';
  import StatesMap from './statesMap.vue';
  import {statesCodes} from './statesCodes.js';

  export default {
    props: {
      groups: Array,
      numbers: Array,
      readonly: {
        type: Boolean
      }
    },
    components: {NumberToBuyItem, StatesMap},
    data () {
      return {
        buyList: [],
        isLoading: false,
        buyProgress: false,
        isAllowed: true,
        lastRequestedNumbersList: [],
        selectedState: '',
        searchValue: '',
        selectedStateName: 'All',
        showSelected: false,
        states: [
          { value: '',   name: 'All' },
          { value: 'AL', name: 'Alabama' },
          { value: 'AK', name: 'Alaska' },
          { value: 'AZ', name: 'Arizona' },
          { value: 'AR', name: 'Arkansas' },
          { value: 'CA', name: 'California' },
          { value: 'CO', name: 'Colorado' },
          { value: 'CT', name: 'Connecticut' },
          { value: 'DE', name: 'Delaware' },
          { value: 'FL', name: 'Florida' },
          { value: 'GA', name: 'Georgia' },
          { value: 'HI', name: 'Hawaii' },
          { value: 'ID', name: 'Idaho' },
          { value: 'IL', name: 'Illinois' },
          { value: 'IN', name: 'Indiana' },
          { value: 'IA', name: 'Iowa' },
          { value: 'KS', name: 'Kansas' },
          { value: 'KY', name: 'Kentucky' },
          { value: 'LA', name: 'Louisiana' },
          { value: 'ME', name: 'Maine' },
          { value: 'MD', name: 'Maryland' },
          { value: 'MA', name: 'Massachusetts' },
          { value: 'MI', name: 'Michigan' },
          { value: 'MN', name: 'Minnesota' },
          { value: 'MS', name: 'Mississippi' },
          { value: 'MO', name: 'Missouri' },
          { value: 'MT', name: 'Montana' },
          { value: 'NE', name: 'Nebraska' },
          { value: 'NV', name: 'Nevada' },
          { value: 'NH', name: 'New Hampshire' },
          { value: 'NJ', name: 'New Jersey' },
          { value: 'NM', name: 'New Mexico' },
          { value: 'NY', name: 'New York' },
          { value: 'NC', name: 'North Carolina' },
          { value: 'ND', name: 'North Dakota' },
          { value: 'OH', name: 'Ohio' },
          { value: 'OK', name: 'Oklahoma' },
          { value: 'OR', name: 'Oregon' },
          { value: 'PA', name: 'Pennsylvania' },
          { value: 'RI', name: 'Rhode Island' },
          { value: 'SC', name: 'South Carolina' },
          { value: 'SD', name: 'South Dakota' },
          { value: 'TN', name: 'Tennessee' },
          { value: 'TX', name: 'Texas' },
          { value: 'UT', name: 'Utah' },
          { value: 'VT', name: 'Vermont' },
          { value: 'VA', name: 'Virginia' },
          { value: 'WA', name: 'Washington' },
          { value: 'WV', name: 'West Virginia' },
          { value: 'WI', name: 'Wisconsin' },
          { value: 'WY', name: 'Wyoming' }
        ]
      }
    },
    destroyed () {
      eventHub.$off('buyList:update', this.updateBuyList);
    },
    watch: {
      selectedState() {
        this.requestNumbers(this.selectedState)
      },
      numbers() {
        console.log('aassssssss')
        this.numbers.forEach((num) => { 
          statesCodes.forEach((state) => {
            if (state.code === num.value.slice(2,5)) {
              num.state = state.state
            } 
          })
        });
      }
    },
    computed: {
      numbersFilteredByState() {
        console.log('nu vot blya', this.numbers)
        if (this.selectedState.name === '' || this.selectedState.name === 'All') return this.numbers;
        return this.numbers.filter((num) => num.state === this.selectedState.name)
      },
      allFilteredNumbers () {
        return _.filter(this.selectedNumbersToBuy, n => this.avilableBySearch(n)); /*fix orthographic error in avilableBySearch*/
      },
      availableNumbers () {
        return _.concat(this.numbers, ...this.groups.map(group => group.numbers));
      },
      buyfilterButtonText () {
        return `Show  ${
          this.hasItemsInBuyList && !this.showSelected
          ? `selected (${this.buyList.length})`
          : 'all'}`;
      },
      filteredNumbers () {
        
        return this.showSelected
        ? this.buyList
        : this.allFilteredNumbers
      },
      hasItemsInBuyList () {
        return this.buyList.length !== 0;
      },
      numbersAvailableToBuy () {
        return this.lastRequestedNumbersList.filter(num => num.phoneNumber !== _.get(_.find(this.availableNumbers, {value: num.phoneNumber}), 'value', undefined))
      },
      selectedNumbersToBuy () {
        return this.selectedStateName !== 'All'
                ? _.filter(this.numbersAvailableToBuy, x => x.region === this.selectedState.value)
                : this.numbersAvailableToBuy;
      },
      selectedState () {
       
        //return _.find(this.states, x => x.name === this.selectedStateName);
      },
      statesNames () {
        return _.map(this.states, x => x.name);
      },
      totalPrice () {
        return _.reduce(this.buyList, (sum, x) => sum + x.price, 0);
      }
    },
    methods: {
      avilableBySearch (n) {
        const parts = n.phoneNumber.match(/\+?(\w+)/gi);
        const number = parts.shift();
        const query = this.searchValue.toLowerCase();
        return number.indexOf(query) > -1
                || _.some(parts, a => a.toLowerCase().indexOf(query) === 0)
                || parts.join(' ').toLowerCase().indexOf(query) === 0;
      },
      buyNumbers () {
        // todo: real buy
        this.buyProgress = true;
        this.$http.post(
          this.$flow.gatewayUrl('identifiers', this.$flow.providersAccountId()),
          {
            identifiers: this.buyList
          },
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`
            }
          }
        )
        .then(() => {
          eventHub.$emit('update numbers data')
          this.buyList = [];
          this.buyProgress = false;
          this.$refs.modal.close();
        })
        .catch((e) => {
          if (e.status == '403') {
            this.isAllowed = false;
          }
          this.buyProgress = false;
        });
      },
      clearSearch () {
        this.searchValue = '';
      },
      changeBuyFilter () {
        //this.selectedStateName = 'All';
        this.showSelected = !this.showSelected;
      },
      openModal() {
        this.$refs.modal.open();
      },
      requestNumbers (selectedState) {
        this.isLoading = true;
        this.showSelected = false;

        this.$http.get(
          this.$flow.gatewayUrl('obtainable-identifiers', this.$flow.providersAccountId()),
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`
            },
            params: {
              region: selectedState ? selectedState.value : ''
            }
          }
        )
        .then(response => response.json())
        .then(d => {
          this.lastRequestedNumbersList = d;
          this.availableNumbers = _.concat(this.numbers, ...this.groups.map(group => group.numbers));
          this.numbersAvailableToBuy = d.filter(num => num.phoneNumber !== _.get(_.find(this.availableNumbers, {value: num.phoneNumber}), undefined));
          this.isLoading = false;
        });

      },
      updateBuyList (newBuylist) {
        this.buyList = newBuylist;
      }
    },
    created () {
      this.requestNumbers();
    
    },
    mounted() {
      console.log('buyModal this.numbers', this.numbers)
    }
  }
</script>

<style lang="scss" scoped rel="stylesheet/scss">
  .numbers-container {
    flex: 1;
  }

  .state-numbers {
    list-style: none;
    height: 100px;
    overflow-y: scroll;
  }

  .buyModal .ui-modal__container {
    width: 55rem;

    .ui-modal__body {
    display: flex;
  }
  }
</style>


