<script lang="ts">
  import * as L from 'leaflet';
  import Leaflet from './lib/map/Leaflet.svelte';
  // import all_states from './lib/all_states.geojson';
  import TileLayer from './lib/map/TileLayer.svelte';
  import StateData from './lib/StateData.svelte';
  import CountyData from './lib/CountyData.svelte';
  import Radio from './lib/Radio.svelte';
  import BarChart from './lib/BarChart.svelte';
  import Bar from './lib/Bar.svelte';
  import PieChart from './lib/PieChart.svelte';
  import Pie from './lib/Pie.svelte';
  import { get_geojson } from './store.js';
  import { onMount } from 'svelte';

  import { csvParseRows } from 'd3-dsv';
  
  import type { Feature, Point } from 'geojson';
  import { feature } from 'topojson-client';



  let data = [];
  let headings = null;
  let selected_category: [string,number];
  let dict = new Map();
  let counties = [];
  let selected_state;
  let selected_county;
  
  let map: Leaflet
  // let map = new L.Map
  const initialBounds = L.latLngBounds([42.99194,-85.72618],[42.0494,-64.78796]);
  let loaded: Boolean = false;
  let topoData: Feature<Point, { [name: string]: any; }>;
  // const urls = ["https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/CT-09-connecticut-counties.json","https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/MA-25-massachusetts-counties.json"];
  


  onMount(async function() {
      const response = await fetch("states.json");
      const json = await response.json();
      console.log(111)
      console.log(response);
      topoData = json.features;
      console.log(topoData)
      const rawdata = await (await fetch('water.csv')).text();
      // // console.log(rawdata)
      data = csvParseRows(rawdata);

  });


  let this_geojson;

  get_geojson.subscribe(value => {
    console.log(value)
    this_geojson = value;
  });

  console.log(this_geojson)

  const backToDefault = () => {
    selected_category = null;
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


  async function updateTopoData(){
      const response = await fetch(this_geojson);
      const json = await response.json();
      // console.log(json);
      if(selected_state != null){
        if (selected_state == 'Connecticut'){
          console.log(this_geojson)
          topoData = feature(json, json.objects.cb_2015_connecticut_county_20m);
        }else if (selected_state == 'Vermont'){
          topoData = feature(json, json.objects.cb_2015_vermont_county_20m);
        }else if (selected_state == 'Rhode Island'){
          topoData = feature(json, json.objects.cb_2015_rhode_island_county_20m);
        }else if (selected_state == 'New York'){
          topoData = feature(json, json.objects.cb_2015_new_york_county_20m);
        }else if (selected_state ==  'Massachusetts'){
          topoData = feature(json, json.objects.cb_2015_massachusetts_county_20m);
        }else if (selected_state == 'New Hampshire'){
          topoData = feature(json, json.objects.cb_2015_new_hampshire_county_20m);
        }else if (selected_state == 'New Jersey'){
          topoData = feature(json, json.objects.cb_2015_new_jersey_county_20m);
        }else if (selected_state == 'Pennsylvania'){
          topoData = feature(json, json.objects.cb_2015_pennsylvania_county_20m);
        }else{
          topoData = feature(json, json.objects.cb_2015_maine_county_20m);
        }
      }else{
        topoData = json.features;
      }
      
      // console.log(geojson)
      console.log(topoData)
      // console.log(map)
      // map.setZoom(8);

      // let map = new L.Map('map');
      // map.fitBounds([
      //   [-4.8587000, 39.8772333],
      //   [-6.4917667, 39.0945000]
      // ])
    }

    async function backToRegional(){
        selected_state = null;
        get_geojson.update(n => "states.json")
      }
    
    function backToType(){
      selected_category = null;
    }
  



  $:{
    console.log(selected_state);

    updateTopoData();

  }
</script>

<svelte:window on:load={() => (loaded = true)} />
  <div style="width: 1000px; height: 10px;" />
  <div class = "banner-container">
    <a href="https://drought.gov" target="_blank">
      
      <img src="https://nedews.nrcc.cornell.edu/img/nidisLogoT.png" alt="NIDIS">
    </a>
    <h2>Northeast DEWS Water Use Mapper</h2>
    <a href="http://www.nrcc.cornell.edu" target="_blank">
      <img src="https://nedews.nrcc.cornell.edu/img/nrccLogoStackedT.png" alt="NRCC">
    </a>
  </div>


  {#if selected_state}
    {#if selected_county}
      {#if selected_category}
      <h2>Currently viewing: {selected_category[0]} of {typeValue} Water in {selected_county},{selected_state}</h2>
      {:else}
      <h2>Currently viewing: {typeValue} Water in {selected_county},{selected_state}</h2>
      {/if}
    {:else}
      {#if selected_category}
      <h2>Currently viewing: {selected_category[0]} of {typeValue} Water in {selected_state}</h2>
      {:else}
      <h2>Currently viewing: {typeValue} Water in {selected_state}</h2>
      {/if}
    {/if}
  {:else}
    {#if selected_category}
      <h2>Currently viewing: {selected_category[0]} of {typeValue} Water in the Northeast</h2>
    {:else}
    <h2>Currently viewing: {typeValue} Water in the Northeast</h2>
    {/if}
  {/if}
  <!-- <h2>Currently viewing: {typeValue} Water in the Northeast</h2> -->
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
            <!-- {#if selected_state == null} -->
            <!-- {#key selected} -->
            <!-- {#await data} -->
              <StateData
                geojson={topoData}
                bind:selected_category={selected_category} 
                bind:selected_state = {selected_state}
                bind:selected_county = {selected_county}
                data={data}
                bind:type={typeValue}
                >
              </StateData>
            <!-- {/await} -->
            <!-- {/key} -->
              <!-- {:else}
                <CountyData
                    geojson={this_geojson}
                    bind:selected_category={selected_category} 
                    bind:selected_state = {selected_state}
                    data={data}
                    bind:type={typeValue}
                    >
                  </CountyData>
              {/if} -->
          {/if}
          <!-- {#if selected_state}
              <button>Back to Regional View</button>
          {/if} -->

        </Leaflet>
      {/if}

      {#if selected_state != null}
        <button id="mapButton" on:click={backToRegional}>Back to Regional View</button>
      {/if}

      <h3>Check out the breakdown for {typeValue} water usage!</h3>
      <Pie bind:selected_category={selected_category} data={data} bind:type={typeValue} bind:selected_state={selected_state} bind:selected_county={selected_county}></Pie>
      {#if selected_category}
      <button id="pieButton" on:click={backToType}>Back to {typeValue} View</button>
  
      {/if}
    </div>
    
  </div>
  <div>

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

  div.banner-container {
    display: flex;
    flex-direction: row;
    background-color: #f7f4eb;
    line-height: 10px;
    border: 1pt solid #6c5626;
    justify-content: space-around;
    align-items: center;
  }

  div.banner-container a{
      /* position: absolute; */
      /* top: 0;
      left: 0 */
      margin-left:0px
  }


  /* div.banner-container a:last-of-type {
      position: absolute;
      top: 0;
      right: 0
  } */

  div.banner-container a img {
      margin: 2px;
      height: 85px
  }

  div.banner-container h2 {
      vertical-align: middle;
      display: inline-block;
      line-height: normal;
      color: #6c5626
  }

  div.banner-container h2 span {
      color: #daa520
  }

  /* .pie{
    display: flex;
    flex-direction: row;
    align-items: center;
  } */
</style>

