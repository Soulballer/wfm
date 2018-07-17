<template>
  <or-modal
  title=""
  class="buy-modal"
  ref="modal"
  :contain-focus="false"
  @close="isAllowed = true"
  >
    <div class="buy-modal-left">
      <div class="map">
        <span class="map__state">Select state on the map: <span>{{selectedState.name}}</span></span>
        <states-map class="map__svg" :selected-state.sync="selectedState"></states-map>

        <p>Your active numbers</p>
        <ul class="state-numbers scrollbar" :class="{ 'no-scroll': numbersFilteredByState.length < 6}">
          <li :key="number.value" v-for="number in numbersFilteredByState">{{number.value}} <span>{{number.state}}</span></li>
        </ul>
      </div>
    </div>

    <div class="buy-modal-right">
      <div class="buy-modal-right__header">
        <p>Buy a numbers</p><p>in <b>{{selectedState.name === 'All' ? 'All states' : selectedState.name}}</b></p>
      </div>
      
      <or-alert @dismiss="isAllowed = true" type="warning" v-show="!isAllowed">
        Only Admin has permissions to manage account numbers. Please contact your admin or OneReach Support.
      </or-alert>

      <div class="search-box">
        <or-textbox
          class="search-filter" 
          placeholder="Search by numbers" 
          name="searchInput" 
          v-model="searchValue" 
          icon="search"
        ></or-textbox>
        <ui-icon @click.native="clearSearch" class="clear-search" icon="close" v-if="searchValue"></ui-icon>
      </div>

      <div class="numbers-container">
        <ui-progress-linear color="primary" size="24" v-if="isLoading"></ui-progress-linear>
        <div v-else>
          <div
            class="button"
            v-show="hasItemsInBuyList"
            @click="changeBuyFilter"
          >{{buyfilterButtonText}}</div>
          <div class="numbers-list scrollbar">
            <div class="ui-select__empty" v-show="!filteredNumbers.length">
              <p>No matching phone numbers</p>
            </div>
            <number-to-buy-item
              v-for="number in filteredNumbers"
              :number="number"
              :buyList="buyList"
              :key="number.phoneNumber"
              transition="expand">
            </number-to-buy-item>
          </div>

          <div class="pagination">
            <or-button
              color="deafult"
              type="secondary"
              v-for="(list, key) in pagination"
              :id="`${key + 1}button`"
              @click="showPagination"
            >{{key + 1}}
            </or-button>
          </div>
        </div>
      </div>

      <div class="footer">
        <or-button
          color="primary"
          @click="buyNumbers"
          :disabled="!hasItemsInBuyList || buyProgress"
        >
          <span v-if="buyProgress">
            <ui-progress-circular color="white" size="18" type="indeterminate" v-show="buyProgress"></ui-progress-circular>
          </span>
          <span v-else>
            Buy {{hasItemsInBuyList ? `(${buyList.length})` : null}}
          </span>
        </or-button>
        <!-- <div class="total">
          Total: USD {{totalPrice}} mothly
        </div> -->
      </div>
    </div>

  </or-modal> 
</template>

<script>
  import eventHub from './eventHub.js';
  import {statesCodes} from './statesCodes.js';

  import NumberToBuyItem from './numberToBuyItem.vue';
  import StatesMap from './statesMap.vue';

  export default {
    props: {
      groups: Array,
      numbers: Array,
    },
    components: {NumberToBuyItem, StatesMap},

    created () {
      eventHub.$on('buyList:update', this.updateBuyList);
      this.requestNumbers();
    },
    destroyed () {
      eventHub.$off('buyList:update', this.updateBuyList);
    },

    data () {
      return {
        buyList: [],
        isLoading: false,
        buyProgress: false,
        isAllowed: true,
        tempArr: [],
        lastRequestedNumbersList: [],
        selectedState: '',
        searchValue: '',
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
    watch: {
      selectedState() {
        this.requestNumbers(this.selectedState)
      },
      numbers() {
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
      pagination() {
        return Math.ceil(this.lastRequestedNumbersList.length/10)
      },
      numbersFilteredByState() {

        if (this.selectedState.name === '' || this.selectedState.name === 'All') return this.numbers;
        return this.numbers.filter((num) => num.state === this.selectedState.name)
      },
      allFilteredNumbers () {
        return _.filter(this.selectedNumbersToBuy, n => this.availableBySearch(n));
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
        return this.tempArr.length === 0 ? this.lastRequestedNumbersList.filter(num => num.phoneNumber !== _.get(_.find(this.availableNumbers, {value: num.phoneNumber}), 'value', undefined)) : this.tempArr
      },
      selectedNumbersToBuy () {
        return this.selectedState.name !== 'All'
                ? _.filter(this.numbersAvailableToBuy, x => x.region === this.selectedState.value)
                : this.numbersAvailableToBuy;
      },
      totalPrice () {
        return _.reduce(this.buyList, (sum, x) => sum + x.price, 0);
      }
    },
    methods: {
      showPagination(event) {
        let num = parseInt(event.target.offsetParent.id)
        console.log('aaa', parseInt(event.target.offsetParent.id));
        this.tempArr = this.lastRequestedNumbersList.slice((num - 1 )*10, num * 10)
       
        console.log('filterednumbers', this.filteredNumbers.slice((num - 1 )*10, num * 10))
        this.lastRequestedNumbersList.slice((num - 1 )*10, num * 10)
      },
      availableBySearch (n) {
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
    }
  }
</script>

<style lang="scss" scoped rel="stylesheet/scss">
  .buy-modal .ui-modal__container {
    position: relative;

    width: 65rem;

    .ui-modal__header {
      position: absolute;
      top: 0;
      right: 0;
    }

    .ui-modal__body {
      display: flex;
      padding: 0;

      .buy-modal-left {
        flex-flow: 3;
        padding: 72px;

        .map {
          &__state {
            display: block;
            padding-left: 14px;
            margin-bottom: 15px;

            color: #868B93;
            font-size: 16px;
            line-height: 21px;

            span {
              color: #0F232E;
            }
          }

          &__svg {
            margin-bottom: 50px;
          }

          p {
            margin-bottom: 0;
            padding-left: 14px;
            padding-bottom: 14px;

            color: #0F232E;
            font-size: 22px;
            font-weight: 100;
            line-height: 26px;
          }

          .state-numbers {
            height: 150px;
            padding: 0 0 0 14px;
            margin: 0;

            list-style: none;
            overflow-y: scroll;

            li {
              margin-bottom: 10px;

              color: #0F232E;
              line-height: 21px;

              span {
                display: inline-block;
                margin-left: 20px;

                color: #868B93;
              }
            }

            &.no-scroll {
              overflow-y: visible;
            }
          }
        }
      }

      .buy-modal-right {
        position: relative;

        flex-grow: 2;
        padding: 40px 50px 20px;

        background-color: #F6F6F6;

        &__header {
          display: flex;
          justify-content: space-between;
          margin-bottom: 23px;
          margin-top: 15px;

          color: #0F232E;
          font-size: 14px;
          line-height: 26px;

          p {
            margin: 0;
          }

          p:first-child {
            font-size: 22px;
          }
        }

        .search-box {
          margin-bottom: 30px;

          .clear-search {
            z-index: 10;
            background-color: #fff;
          }

          .ui-textbox {
            margin-bottom: 0;
            border: none;
          }

          .ui-textbox__icon-wrapper {
            top: 10px;
            left: inherit;
            right: 5px;

            .ui-icon {
              font-size: 21px;
              color: #91969D !important;
            }
          }

          .ui-textbox__input {
            padding-left: 15px;
          }
        }

        .numbers-container {
          position: relative; 
          padding-top: 22px;
          
          .button {
            position: absolute;
            top: 0;

            padding: 0;

            font-size: 14px;
            line-height: 21px;
          }

          .numbers-list {
            height: 400px;
            max-height: 400px;
            margin-bottom: 25px;

            .ui-select__empty p {
              text-align: center;
            }
          }
        }

        .footer {
          position: absolute;
          bottom: 50px;
          left: 0;
          right: 0;

          display: flex;
          justify-content: center;
        }
      }
    }
  }
</style>


