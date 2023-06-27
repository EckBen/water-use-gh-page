<script lang="ts">
    import { arc, pie } from "d3-shape";
    import {onMount} from 'svelte';
    // import { colorCategoricalDragon } from "$utils/brand";
    import {schemeRdBu, interpolateSpectral} from 'd3-scale-chromatic';
    import * as d3 from 'd3';
    import CountyData from './CountyData.svelte';

    export let data: Array<Array<string>>;
    export let selected: [string,number] = null;
    export let type: String = null;

    // [2, 3, 5, 8, 13, 21]

    // onMount(() => {
    // 	console.log(data)
  	// });

    let loaded = false;

    let pies;
    let colors;
    let clr;
    let pieData;
    let column_names;
    let columns;
    const updateChart = () => {
      
        if (type == "usage"){
          columns = [18,26,35,47,56,65,74,83,97];
          column_names = ["Public Supply","Self-supplied domestic","Industrial","Crop Irrigatin","Golf Course Irrigation","LiveStock","Aquaculture","Mining","thermoelectric consumptive"];
        }else{
          columns = [7,20,29,36,63,68,77];
          column_names = ["Public Supply","Self-supplied domestic","Industrial","Crop Irrigatin","Golf Course Irrigation","LiveStock","Aquaculture","Mining","thermoelectric consumptive"];
        }
        
        let tot;
        let width = 400;
        let height = 300;
        let radius = 150;
        pieData = [];
        for (let idx in columns){
            let tot = 0;
            for (let i = 1; i < data.length; i++){
                tot += Number(data[i][columns[idx]]);
            }
            pieData.push(tot);
            clr = interpolateSpectral(tot*10);
        }
        console.log(pieData);
        // console.log(pieData)

        let arcs = pie()(pieData);
        const arcGen = arc();

        pies = arcs.map((arc) => {
            let input = {
                innerRadius: 10,
                outerRadius: 100,
                startAngle: arc.startAngle,
                endAngle: arc.endAngle,
            };
            return arcGen(input);
        })
        
        colors = ['pink','peachpuff','palevioletred','plum','purple','rebeccapurple','	thistle'];


    }
    
    document.addEventListener("DOMContentLoaded", function(event) {
      console.log("DOM fully loaded and parsed");
    });


    let handleClick = (e) => {
      let selected_col;
      for (let i = 0; i < column_names.length; i++){
        if (column_names[i] == e.currentTarget.id){
          selected_col = i;
          break;
        }
      }
      selected = [e.currentTarget.id,columns[selected_col]]
      

    }

    let handleMouseOver = (e) => {

      let this_use = e.target.id;

      let text_use = d3.select("#water_use");
      text_use.select("#this_use").text(this_use);
      let slices = d3.selectAll('.class-slice');


    }
    let handleMouseOut= (e) => {

      let text_use = d3.select("#water_use");
      text_use.select("#this_use").text("");

    }
  

    $: {
        console.log(data);
        updateChart();
    }
  </script>
  
  <!-- <Plot the svg paths  -->
  <div class = "type">
    {#if type == "source"}
      <h2>Ground Water Sources</h2>
    {/if}
    {#if type == "usage"}
      <h2>Ground Water Usage</h2>
    {/if}
    <svg id ='pie' width="400" height="300">
      {#each pies as pie, i}
        <path
          d={pie}
          class="slice"
          id={String(column_names[i])}
          style="transform:translate(150px, 150px)"
          fill={colors[i]}
          on:click={handleClick}
          on:mouseover={handleMouseOver}
          on:mouseout={handleMouseOut}
        />
      {/each}
    </svg> 
    <table id="table">
      <th id="thead"></th>
      <tr>
        <td id="water_use">
          <p id="this_use"></p>
        </td>
      </tr>
    </table>
  </div>

  <style>
    .type{
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
  </style>
  

  <!-- <button>Click Me!</button> -->

  <!-- <p>This is {selected}</p> -->

  <!-- {#if selected_col != null}
    <CountyData
    geojson={topoData}
    selected={selectedHeading}
    data={data}
    >
  </CountyData>
  {/if} -->




  