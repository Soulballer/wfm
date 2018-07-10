<template>
  <div class="map">
    <p>{{clickedState}}</p>
    <div id="map__tooltip"></div>
    <svg width="960" height="600" class="map__main" id="statesvg"></svg>
  </div>
</template>

<script>
import * as d3 from '../../node_modules/d3/build/d3.js';
import {uStatePaths} from './statePaths.js';

console.log('ss', uStatePaths);

export default {
  name: 'StatesMap',
  props: {
    selectedState: {
      type: Object,
      default() {
        return {id: '', value: ''}
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
      clickedState: '',
      localSelectedState: {},
      localUStates: {},
      uStatePaths: [],
      uStates: {}
    }
  },
  mounted() {
    let self = this;
    this.uStates.draw = function(id, data, toolTip, self){		
      function mouseOver(d){
        d3.select(this).style("fill", 'red');
        d3.select("#tooltip").transition().duration(200).style("opacity", .9);      
        
        d3.select("#tooltip").html(toolTip(d.n, data[d.id]))  
          .style("left", (d3.event.pageX) + "px")  
          .style("top", (d3.event.pageY - 28) + "px");
      }
      
      function mouseOut(){
        d3.select("#tooltip").transition().duration(500).style("opacity", 0);    
        d3.select(this).style("fill",function(d){ return data[d.id].color; })  
      }
      
      function mouseClick(event) {
        console.log('target', event.id)
        self.clickedState = event.id;
      }
      
      d3.select(id).selectAll(".state")
        .data(uStatePaths).enter().append("path").attr("class","state").attr("d",function(d){ return d.d;})
        .style("fill",function(d){ return data[d.id].color; })
        .style("stroke", "black")
        .on("mouseover", mouseOver).on("mouseout", mouseOut).on('click', mouseClick);
    }

    function tooltipHtml(n, d) {	/* function to create html content string in tooltip div. */
		return "<h4>"+n+"</h4><table>"+
			"<tr><td>Low</td><td>"+(d.low)+"</td></tr>"+
			"<tr><td>Average</td><td>"+(d.avg)+"</td></tr>"+
			"<tr><td>High</td><td>"+(d.high)+"</td></tr>"+
			"</table>";
	}
	
	var sampleData ={};	/* Sample random data. */	
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
					avg:Math.round((low+mid+high)/3), color:d3.interpolate("#ffffcc", "#800026")(low/100)}; 
		});
	
	/* draw states on id #statesvg */	
 this.uStates.draw("#statesvg", sampleData, tooltipHtml, self);
	
	d3.select(self.frameElement).style("height", "600px"); 
  }

}
</script>

<style lang="scss" scoped>

</style>


