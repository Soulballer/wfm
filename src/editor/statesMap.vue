<template>
  <div class="map">
    <div id="map__tooltip"></div>
    <svg width="500" height="350" viewBox="0 0 960 600" class="map__main" id="statesvg"></svg>
  </div>
</template>

<script>
import * as d3 from '../../node_modules/d3/build/d3.js';
import {uStatePaths} from './statePaths.js';

export default {
  name: 'StatesMap',
  props: {
    availableNumbers: {
      type: Object
    },
    selectedState: {
      type: Object,
      default() {
        return {id: ''}
      }
    },
    // uStates: {
    //   type: Object,
    //   default() {
    //     return {}
    //   }
    // }
  },

  data() {
    return {
      clickedState: {id: ''},
      localUStates: {},
      localSelectedState: {},
      uStatePaths: [],
      uStates: {},
      sampleData: {},
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
        { value: 'LS', name: 'Louisiana' },
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
  mounted() {
    console.log('mama', this.availableNumbers)
    let self = this;
    this.clickedState = {id: ''}; 
    this.uStates.draw = function(id, data, toolTip, context){		
      function mouseOver(d){
        if (d.id !== context.clickedState) {
        d3.select(this).style("fill", 'orange');
        d3.select("#tooltip").transition().duration(200).style("opacity", .9);      
        
        d3.select("#tooltip").html(toolTip(d.n, data[d.id]))  
          .style("left", (d3.event.pageX) + "px")  
          .style("top", (d3.event.pageY - 28) + "px");
        }
      }
      
      function mouseOut(event){
        if (event.id !== context.clickedState.id) {
          d3.select("#tooltip").transition().duration(500).style("opacity", 0);    
          d3.select(this).style("fill", function(d){ return data[d.id].color; }) 
        }
      }
      
      function mouseClick(event) {
        if (context.clickedState.id !== event.id && Object.keys(context.localSelectedState).length !== 0) {
          d3.select(context.localSelectedState).style("fill", function(d){ return data[d.id].color; }) 
        } else if (context.clickedState.id === event.id) {
          d3.select(this).style("fill", function(d){ return data[d.id].color; }) 
          context.localSelectedState = '';
          context.clickedState = {id: ''};
          return
        }
  
        context.clickedState = {id: event.id};
        context.localSelectedState = this
        d3.select(this).style("fill", 'orange');
      }
      
      d3.select(id).selectAll(".state")
        .data(uStatePaths).enter().append("path").attr("class","state").attr("d",function(d){ return d.d;})
        .style("fill",function(d){ return data[d.id].color; })
        .style("stroke", "black")
        .on("mouseover", mouseOver).on("mouseout", mouseOut).on('click', mouseClick);
        d3.select(id).select('svg').selectAll(".state").selectAll('path').style('vector-effect', 'non-scaling-stroke');
    }
    function tooltipHtml(n, d) {	/* function to create html content string in tooltip div. */
		return "<h4>"+n+"</h4><table>"+
			"<tr><td>Low</td><td>"+(d.low)+"</td></tr>"+
			"<tr><td>Average</td><td>"+(d.avg)+"</td></tr>"+
			"<tr><td>High</td><td>"+(d.high)+"</td></tr>"+
			"</table>";
	}
	let sampleData = {};	/* Sample random data. */	
	["HI", "AK", "FL", "SC", "GA", "AL", "NC", "TN", "RI", "CT", "MA",
	"ME", "NH", "VT", "NY", "NJ", "PA", "DE", "MD", "WV", "KY", "OH", 
	"MI", "WY", "MT", "ID", "WA", "DC", "TX", "CA", "AZ", "NV", "UT", 
	"CO", "NM", "OR", "ND", "SD", "NE", "IA", "MS", "IN", "IL", "MN", 
	"WI", "MO", "AR", "OK", "KS", "LS", "VA"]
		.forEach(function(d){ 
			var low=Math.round(100*Math.random()), 
				mid=Math.round(100*Math.random()), 
				high=Math.round(100*Math.random());
			sampleData[d]={low:d3.min([low,mid,high]), high:d3.max([low,mid,high]), 
					avg:Math.round((low+mid+high)/3), color:d3.interpolate("#D6EBD4", "#174F15")(low/100)}; 
    });
    
    this.sampleData = sampleData;
	
  /* draw states on id #statesvg */	
 this.uStates.draw("#statesvg", this.sampleData, tooltipHtml, self);
	
	d3.select(self.frameElement).style("height", "600px"); 
  },
  watch: {
    availableNumbers() {
      let self = this;
      if (this.availableNumbers) {
        let stateCounter = this.availableNumbers.reduce((obj, num) => {
          console.log('!!!', obj, num.state)
          obj[num.state] ? obj[num.state] += 1 : obj[num.state] = 1;
          obj.max = obj.max ? obj.max > obj[num.state] ? obj.max : obj[num.state] : obj[num.state];
          return obj;
        }, {})



        console.log('result', stateCounter)
      }
      d3.select('#statesvg').selectAll('.states').remove();
      this.uStates.draw("#statesvg", this.sampleData, null, self);
    },
    clickedState() {
      //console.log('clickedState', this.clickedState, ...this.states.filter((state) => state.value == this.clickedState.id));
      this.$emit('update:selectedState', ...this.states.filter((state) => state.value == this.clickedState.id));
    }
  }

}
</script>

<style lang="scss" scoped>

</style>


