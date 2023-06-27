<script lang="ts">
    import * as L from 'leaflet';
    import Leaflet from './lib/map/Leaflet.svelte';
    // import all_states from './all_states.geojson';
    import TileLayer from './lib/map/TileLayer.svelte';
    import StateData from './lib/StateData.svelte';
    import CountyData from './lib/CountyData.svelte';
    import Radio from './lib/Radio.svelte';
    import BarChart from './lib/BarChart.svelte';
    import Bar from './lib/Bar.svelte';
    import PieChart from './lib/PieChart.svelte';
    import Pie from './lib/Pie.svelte';
    import { onMount } from 'svelte';
  
    import { csvParseRows } from 'd3-dsv';
    
    import type { Feature, Point } from 'geojson';
    import { feature } from 'topojson-client';
  
    let data = [];
    let headings = null;
    let selected: [string,number];
    let dict = new Map();
    let counties = [];
    let state;
    
    let map: Leaflet
    const initialBounds = L.latLngBounds([40.99194,-73.72618],[42.0494,-71.78796]);
    let loaded: Boolean = false;
    let topoData: Feature<Point, { [name: string]: any; }>;
    // const urls = ["https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/CT-09-connecticut-counties.json","https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/MA-25-massachusetts-counties.json"];
    
    onMount(async function() {
        const response = await fetch(
          
          "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/CT-09-connecticut-counties.json"
        );
        const json = await response.json();
        // console.log(all_states);
        // topoData = all_states.properties;
        topoData = feature(json, json.objects.cb_2015_connecticut_county_20m)
        // console.log(topoData);
  
        let state = 'CT';
        const rawdata = await (await fetch('water.csv')).text();
        // console.log(rawdata)
        data = csvParseRows(rawdata);
        // data = data.filter(d => d['STATE'] === 'CT')
        let temp = [];
        getState();
        function getState(){
          data.forEach(d => {
            if (d[0] == state){
              temp.push(d);
            }
          })
        }
        data = temp;
        console.log(data);
  
        for (let i = 0; i < data.length; i++){
          counties.push(data[i][4]);
        }
        const rawHeadings = await (await fetch('water_description.csv')).text();
        headings = csvParseRows(rawHeadings).map((row, idx) => [row[1],idx]);
  
    });
  
    let clicked = false;
    function addBarChart(){
      clicked = true;
    }
  
  // $: if (selectedHeading || selectedHeading != selectedHeading) { 
  //   let color = ['yellow', 'orange', 'red'];
  //   for (let i = 0; i < data.length; i++){
  //     let key = data[i][4];
  //     let val = data[i][selectedHeading[1]];
  //     let val2;
  //     if (val > 5){
  //       val2 = color[2];
  //     }else if(val > 2.5){
  //       val2 = color[1];
  //     }else{
  //       val2 = color[0];
  //     }
  //     dict.set(key, val2);
  //   }
  //   console.log(dict);
  
  // }
  
  const backToDefault = () => {
    selected = null;
  }
  
  let typeValue;
  
  const options = [{
    value:'ground',
    label: 'ground water'
  },{
    value: 'surface',
    label: 'surface water'
  }]
  
  
  const max = 1;
  let selected_box = ['ground'];
  let reload = 0;
  // let checked = options.map(() => false);
  
  function handleCheckBox(e){
    // type = e.target.__value;
    // reload++;
    // console.log(reload);
  
  }
  
  
  </script>
  
  <div>
    
  </div>
  <svelte:window on:load={() => (loaded = true)} />
  <div style="width: 1000px; height: 10px;" />
  <!-- {#if headings}
  <select bind:value={selectedHeading}>
    {#each headings.slice(6,headings.length-2) as heading}
      <option value={heading}>
        {heading[0]}
      </option>
    {/each}
  </select>
  {/if} -->
  
  <div class = "all">
    <h2>Explore water usage in the Northeast!</h2>
    <Radio {options} legend = 'select a type!' bind:userSelected = {typeValue} />
    <div class="w-screen h-screen" id="container">
      
      <div style="grid-area:map;">
        <!-- Show the map only once the window has loaded, so that Leaflet gets the sizing right. -->
        {#if loaded || document.readyState === 'complete'}
          <Leaflet bind:this={map} bounds={initialBounds}>
            <TileLayer
              url={"https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}@2x.png"} 
              options = {{attribution: 'Data Source: <a href="https://www.rcc-acis.org/">ACIS</a>'}}
            />
            {#if topoData}
              <!-- {#key selected} -->
              <!-- {#await data} -->
              <CountyData
                geojson={topoData}
                bind:selected={selected} 
                bind:state = {state}
                data={data}
                bind:type={typeValue}
                >
              </CountyData>
              <!-- {/await} -->
              <!-- {/key} -->
            {/if}
          </Leaflet>
        {/if}
      </div>
      
    </div>
    <!-- <input type="checkbox" checked={selected_box} id='ground' disabled={selected_box.length == 1 && !selected_box.includes('ground')} on:change={handleCheckBox}>
    <label>Ground Water</label>
    <input type="checkbox" checked={selected_box} id='surface' disabled={selected_box.length == 1 && !selected_box.includes('surface')} on:change={handleCheckBox}>
    <label>Surface Water</label> -->
    <!-- {#each options as option, index}
      <div class="options">
        
        <input type = "checkbox" bind:group={selected_box} name="options" value={option} id="option{index}" disabled={selected_box.length == max && !selected_box.includes(option)} on:input={handleCheckBox}>
        <label for="option{index}">{option}</label>
  
      </div>
    {/each} -->
    <Pie bind:selected={selected} data={data} bind:type={typeValue} ></Pie>
    <p></p>
    <!-- <label>
      <input type="checkbox" bind:checked={ground_clicked}>
      Ground Water Source
      <input type="checkbox" bind:checked={surface_clicked}>
    </label>
  
    {#if ground_clicked}
      <p>Retrieving ground water source data...</p>
    {/if}
    {#if ground_clicked}
      <p>Retrieving ground water source data...</p>
    {/if} -->
    <!-- <div>
      <BarChart data={data}> </BarChart>
    </div> -->
    
    
    <!-- <div class = "pie">
      <div>
          <PieChart bind:selected={selected} data={data} type={"source"}></PieChart>
      </div>
      <div>
          <PieChart bind:selected={selected} data={data} type={"usage"}></PieChart>
      </div>
    </div> -->
  
    <!-- <div class = "pie"> -->
      <!-- <div>
          <Pie bind:selected={selected} data={data} type={"source"}></Pie>
      </div> -->
      <!-- <div>
          <Pie bind:selected={selected} data={data} type={"usage"}></Pie>
      </div>
    </div> -->
    
    
    {#if selected != null}
      <!-- <p>{selected}</p> -->
      <button on:click={backToDefault}>Reset</button>
      <!-- <CountyData
                geojson={topoData}
                bind:selected={selected}
                data={data}
      >
      </CountyData> -->
    {/if} 
  </div>
  
  
  <style>
    .all{
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
  
    #container {
      width: 100%;
      height: 600px;
      display: grid;
      grid-template:
        'map' auto;
    }
  
    /* .pie{
      display: flex;
      flex-direction: row;
      align-items: center;
    } */
  </style>
  