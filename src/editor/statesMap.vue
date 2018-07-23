<template>
  <div class="map">
    <div class="map__legend">
      <div :key="color.color" v-for="color in colorPalette">
        <div class="color-box" :style="{'background-color': color.color}"></div>
        <span>{{color.min}} {{color.max == Infinity ? '+' : `- ${color.max}`}}</span>
      </div>
      <p>Population per square<br> mile by state.<br> 2015 census figures.</p>
    </div>
    <svg width="500" height="350" viewBox="0 0 960 600" id="statesvg"></svg>
  </div>
</template>

<script>
  import eventHub from './eventHub.js';
  import {uStatePaths} from './statePaths.js';

  import * as d3 from 'd3';

  export default {
    name: 'StatesMap',
    props: {
      selectedState: {
        type: Object,
        default() {
          return {id: '', name: 'All'}
        }
      }
    },

    computed: {
      colorData() {
        let tempData = {};
        _.forEach(this.states, (state) => { tempData[state.value] = {color: this.getColor(state).color} });

        return tempData;
      }
    },
    data() {
      return {
        clickedState: {id: ''},
        colorPalette: [
          { color: '#0C280B', min: 1000, max: Infinity },
          { color: '#174F15', min: 500,  max: 1000 },
          { color: '#21781F', min: 200,  max: 500 },
          { color: '#4EB748', min: 100,  max: 200 },
          { color: '#8ECB84', min: 50,   max: 100 },
          { color: '#B1DAAB', min: 20,   max: 50 },
          { color: '#D6EBD4', min: 0,    max: 20 }
        ],
        states: [
          { value: '',   name: 'All', population: 0 },
          { value: 'AL', name: 'Alabama', population: 95 },
          { value: 'AK', name: 'Alaska', population: 1 },
          { value: 'AZ', name: 'Arizona', population: 60 },
          { value: 'AR', name: 'Arkansas', population: 57 },
          { value: 'CA', name: 'California', population: 251 },
          { value: 'CO', name: 'Colorado', population: 52 },
          { value: 'CT', name: 'Connecticut', population: 741 },
          { value: "DC", name: "Washington DC", population: 11011 },
          { value: 'DE', name: 'Delaware', population: 485 },
          { value: 'FL', name: 'Florida', population: 378 },
          { value: 'GA', name: 'Georgia', population: 177 },
          { value: 'HI', name: 'Hawaii', population: 222 },
          { value: 'ID', name: 'Idaho', population: 20 },
          { value: 'IL', name: 'Illinois', population: 231 },
          { value: 'IN', name: 'Indiana', population: 184 },
          { value: 'IA', name: 'Iowa', population: 55 },
          { value: 'KS', name: 'Kansas', population: 36 },
          { value: 'KY', name: 'Kentucky', population: 112 },
          { value: 'LS', name: 'Louisiana', population: 108 },
          { value: 'ME', name: 'Maine', population: 43 },
          { value: 'MD', name: 'Maryland', population: 618 },
          { value: 'MA', name: 'Massachusetts', population: 871 },
          { value: 'MI', name: 'Michigan', population: 175 },
          { value: 'MN', name: 'Minnesota', population: 68 },
          { value: 'MS', name: 'Mississippi', population: 63 },
          { value: 'MO', name: 'Missouri', population: 88 },
          { value: 'MT', name: 'Montana', population: 7 },
          { value: 'NE', name: 'Nebraska', population: 24 },
          { value: 'NV', name: 'Nevada', population: 26 },
          { value: 'NH', name: 'New Hampshire', population: 148 },
          { value: 'NJ', name: 'New Jersey', population: 1218 },
          { value: 'NM', name: 'New Mexico', population: 17 },
          { value: 'NY', name: 'New York', population: 420 },
          { value: 'NC', name: 'North Carolina', population: 206 },
          { value: 'ND', name: 'North Dakota', population: 10 },
          { value: 'OH', name: 'Ohio', population: 284 },
          { value: 'OK', name: 'Oklahoma', population: 57 },
          { value: 'OR', name: 'Oregon', population: 41 },
          { value: 'PA', name: 'Pennsylvania', population: 286 },
          { value: 'RI', name: 'Rhode Island', population: 1021 },
          { value: 'SC', name: 'South Carolina', population: 162 },
          { value: 'SD', name: 'South Dakota', population: 11 },
          { value: 'TN', name: 'Tennessee', population: 160 },
          { value: 'TX', name: 'Texas', population: 105 },
          { value: 'UT', name: 'Utah', population: 36 },
          { value: 'VT', name: 'Vermont', population: 67 },
          { value: 'VA', name: 'Virginia', population: 212 },
          { value: 'WA', name: 'Washington', population: 107 },
          { value: 'WV', name: 'West Virginia', population: 76 },
          { value: 'WI', name: 'Wisconsin', population: 106 },
          { value: 'WY', name: 'Wyoming', population: 6 }
        ]
      }
    },
    methods: {
      draw(id, data) {
        d3.select(id).selectAll(".state")
          .data(uStatePaths).enter().append("path").attr("id", d => d.id).attr("d", d => d.d)
          .style("fill", d => data[d.id].color).style("stroke", "black")
          .on("mouseover", this.mouseOver).on("mouseout", this.mouseOut).on('click', this.mouseClick);
      },
      getColor(state) {
        return [..._.filter(this.colorPalette, (data) => data.max > state.population && data.min <= state.population)][0];
      },
      mouseClick(event) {
        if (this.clickedState.id !== event.id && !_.isEmpty(this.clickedState.id)) {
          d3.select(`#${this.clickedState.id}`).style("fill", this.colorData[event.id].color) ;
        } else if (this.clickedState.id === event.id){
          d3.select(`#${event.id}`).style("fill", this.colorData[event.id].color) 
          this.clickedState = {id: ''};
          return
        }

        this.clickedState = {id: event.id};
        d3.select(`#${event.id}`).style("fill", 'orange');
      },
      mouseOver(event) {
        return event.id !== this.clickedState.id ? d3.select(`#${event.id}`).style("fill", 'orange') : '';
      },
      mouseOut(event) {
        return event.id !== this.clickedState.id ? d3.select(`#${event.id}`).style("fill", this.colorData[event.id].color) : ''; 
      }
    },
    mounted() {
      this.draw("#statesvg", this.colorData);
    },
    watch: {
      clickedState() {
        this.$emit('update:selectedState', ..._.filter(this.states, (state) => state.value === this.clickedState.id));
        eventHub.$emit('update state');
      }
    }
  }
</script>

<style lang="scss" scoped>
  .map {
    position: relative;
  }

  .map__legend {
    position: absolute;
    bottom: 45px;
    right: -25px;

    font-size: 0px;

    .color-box {
      display: inline-block;
      width: 15px;
      height: 10px;
      margin-right: 3px;

      border: 1px solid black;
      border-top: 0px;

      vertical-align: top;
    }

    span {
      display: inline-block;

      font-size: 9px;
      line-height: 10px;

      vertical-align: top;
    }

    p {
      margin-top: 5px;
      margin-bottom: 0;
      padding: 0;

      font-size: 9px;
      line-height: normal;
    }
  }
</style>


