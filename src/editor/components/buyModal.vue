<template>
  <or-modal
    title=""
    class="buy-modal"
    ref="modal"
    :contain-focus="false"
    @open="requestNumbers"
    @close="isAllowed = true"
  >
    <div class="buy-modal-left">
      <div class="map">
        <span class="map__state">Select state on the map: <span>{{selectedState.name}}</span></span>
        <states-map class="map__svg" :selected-state.sync="selectedState"></states-map>

        <p>Your active numbers</p>
        <ul class="state-numbers scrollbar" :class="{ 'no-scroll': numbersFilteredByState.length < numbersAvailableToShow}">
          <li :key="number.id" v-for="number in numbersFilteredByState">{{number.value}} <span>{{number.state}}</span></li>
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
              v-for="number in splitToPagesNumbers"
              :number="number"
              :buyList="buyList"
              :key="number.phoneNumber"
              transition="expand">
            </number-to-buy-item>
          </div>

          <div class="pagination">
            <or-button
              :key="key"
              :class="{selected: pageNumber  === key + 1}"
              color="deafult"
              type="secondary"
              v-for="(list, key) in paginationButtons"
              :id="`${key + 1}button`"
              @click="changePageNumber"
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
  import eventHub from '../helpers/eventHub.js';
  import {usCodes} from './mapData/usAreaCodes.js';

  import NumberToBuyItem from './numberToBuyItem.vue';
  import StatesMap from './statesMap.vue';

  export default {
    props: {
      numbers: {
        type: Array,
        default() {
          return []
        }
      }
    },
    components: { NumberToBuyItem, StatesMap },

    created () {
      eventHub.$on('buyList:update', this.updateBuyList);
      eventHub.$on('update state', this.requestNumbers);
    },
    destroyed () {
      eventHub.$off('buyList:update', this.updateBuyList);
      eventHub.$off('update state', this.requestNumbers);
    },

    computed: {
      allFilteredNumbers () {
        return _.filter(this.numbersAvailableToBuy, n => this.availableBySearch(n));
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
      mappedNumbers() {
        return _
          .chain(this.numbers)
          .filter(num => !num.isGroup)
          .reduce((arr, num) => {
            arr.push({id: num.id, value: num.phoneNumber, state: this.getState(num.phoneNumber)});
            return arr
          }, [])
          .value();
      },
      numbersFilteredByState() {
        if (this.selectedState.name === 'All') return this.mappedNumbers;

        return _.filter(this.mappedNumbers, num => num.state === this.selectedState.name)
      },
      paginationButtons() {
        let pagesLength = Math.ceil(this.filteredNumbers.length / this.numbersToShow);
        if (this.pageNumber > pagesLength) this.pageNumber = pagesLength || 1 ;

        return pagesLength;
      },
      splitToPagesNumbers () {
        return _.slice(this.filteredNumbers, (this.pageNumber - 1 ) * this.numbersToShow, this.pageNumber * this.numbersToShow);
      },
      totalPrice () {
        return _.reduce(this.buyList, (sum, x) => sum + x.price, 0);
      }
    },
    data () {
      return {
        buyList: [],
        buyProgress: false,
        isAllowed: true,
        isLoading: false,
        pageNumber: 1,
        numbersToShow: 10,
        numbersAvailableToShow: 6,
        numbersAvailableToBuy: [],
        selectedState: {id: '', name: 'All'},
        searchValue: '',
        showSelected: false
      }
    },
    methods: {
      availableBySearch (n) {
        const parts = n.phoneNumber.match(/\+?(\w+)/gi);
        const number = parts.shift();
        const query = _.toLower(this.searchValue);
        return number.indexOf(query) > -1
                || _.some(parts, a => _.toLower(a).indexOf(query) === 0)
                || _.toLower(parts.join(' ')).indexOf(query) === 0;
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
          eventHub.$emit('update numbers data');
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
      changePageNumber(event) {
        if (!event) return;

        this.pageNumber = _.parseInt(event.target.offsetParent.id)
      },
      getState(num) {
        return usCodes[num.slice(2,5)].state;
      },
      openModal() {
        this.$refs.modal.open();
      },
      requestNumbers (selectedState) {
        this.isLoading = true;
        this.pageNumber = 1;
        this.showSelected = false;
      
        this.$http.get(
          this.$flow.gatewayUrl('obtainable-identifiers', this.$flow.providersAccountId()),
          {
            headers: {
              Authorization: `USER ${this.$settings.token}`
            },
            params: {
              region: this.selectedState ? this.selectedState.value : ''
            }
          }
        )
        .then(response => response.json())
        .then(num => {
          this.numbersAvailableToBuy = num
          this.isLoading = false;
        });
      },
      updateBuyList (newBuylist) {
        this.buyList = newBuylist;
      }
    }
  }
</script>

<style lang="scss" scoped>
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

          & > p {
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
            height: 340px;
            max-height: 340px;
            margin-bottom: 10px;

            .ui-select__empty p {
              text-align: center;
            }
          }

          .pagination {
            display: flex;
            justify-content: center;

            .ui-button {
              width: 32px;
              min-width: auto;
              height: 32px;
              padding: 16px;

              line-height: 0;

              border: none;
              border-radius: 50%;

              &.selected {
                .ui-button__content {
                  color: #64B2DA;
                }
              }

              &__content {
                color: #6D6D6D;
              }
            }
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
</style>


