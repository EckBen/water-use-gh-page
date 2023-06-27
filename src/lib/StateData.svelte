<!-- TODO: set zoom to bounding box -->
<script lang="ts">
    import * as L from 'leaflet';
    import {schemeRdBu, interpolateSpectral, interpolateYlOrRd, interpolateGnBu} from 'd3-scale-chromatic';
    import {
      getContext,
      setContext,
      onDestroy,
      createEventDispatcher,
      onMount,
    } from 'svelte';
    import * as d3 from 'd3';
    import CountyData from './CountyData.svelte';
    import { feature } from 'topojson-client';
    import { get_geojson } from '../store.js';
  
    const dispatch = createEventDispatcher();
  
    export let geojson: any;
    export let selected_category: [string,number] = null;
    export let data: Array<Array<string>> = null;
    export let type: string = null;
    export let selected_state: string = null;
    export let selected_county: string = null;
  
    const container = getContext<() => L.LayerGroup>('layerGroup')();
    
    let layer: L.GeoJSON = null;

    function update_geojson(){
      let this_state_counties;
      if (selected_state == 'Connecticut'){
          this_state_counties = "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/CT-09-connecticut-counties.json";
        }else if (selected_state == 'Vermont'){
          this_state_counties = "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/VT-50-vermont-counties.json";
        }else if (selected_state == 'Rhode Island'){
          this_state_counties = "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/RI-44-rhode-island-counties.json";
        }else if (selected_state == 'New York'){
          this_state_counties = "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/NY-36-new-york-counties.json";
        }else if (selected_state ==  'Massachusetts'){
          this_state_counties = "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/MA-25-massachusetts-counties.json";
        }else if (selected_state == 'New Hampshire'){
          this_state_counties = "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/NH-33-new-hampshire-counties.json";
        }else if (selected_state == 'New Jersey'){
          this_state_counties = "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/NJ-34-new-jersey-counties.json";
        }else if (selected_state == 'Pennsylvania'){
          this_state_counties = "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/PA-42-pennsylvania-counties.json";
        }else{
          this_state_counties = "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/ME-23-maine-counties.json";
        }
      get_geojson.update(n => this_state_counties)
    }

    let data_ready;
      let corr_ground = new Map();
      let corr_surface = new Map();
      let corr_total = new Map();
      let ground = 0;
      let surface = 0;
      let total = 0;
      let ptr_ground = {}
      let ptr_surface = {}
      let ptr_total = {}

      let state_to_url = {
        'CT': "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/CT-09-connecticut-counties.json",
        'RI': "https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/RI-44-rhode-island-counties.json"
      }

      let state_abbre_to_state = {
        'CT' : "Connecticut",
        'RI' : 'Rhode Island',
        'NY' : 'New York',
        'MA' : 'Massachusetts',
        'NH' : 'New Hampshire',
        'NJ' : 'New Jersey',
        'PA' : 'Pennsylvania',
        'VT' : 'Vermont',
        'ME' : 'Maine'
      }

      let state_to_state_abre = {
         "Connecticut":'CT',
        'Rhode Island':'RI',
        'New York':'NY',
        'Massachusetts': 'MA',
        'New Hampshire': 'NH',
        'New Jersey': 'NJ',
        'Pennsylvania' : 'PA',
        'Vermont': 'VT',
        'Maine': 'ME'
      }

      // const drawCounties(async function()){

      // }

      // async function drawCounties(){
      //   const response = await fetch("https://raw.githubusercontent.com/deldersveld/topojson/master/countries/us-states/CT-09-connecticut-counties.json");
      //   const json = await response.json();
      //   console.log(json);
      //   const topoData = feature(json, json.objects.cb_2015_connecticut_county_20m)
      // }

      let ptr_selected = {};
      let corr_selected = new Map();
      
  
      const getData = () => {
        console.log(data.length)
        if (data == null) return;
        // console.log(typeof data[1])
        // corr_ground = new Map();
        // corr_surface = new Map();
        // corr_total = new Map();
        // ground = 0;
        // surface = 0;
        // total = 0;
        // ptr_ground = {}
        // ptr_surface = {}
        // ptr_total = {}

        if(selected_state == null){
          if (selected_category != null){
            let curr_column = selected_category[1];
            let prev_state = 'CT';
            let idx = 2;
            while (idx < data.length - 1){
              prev_state = data[idx][0];
              let selected_total = 0;
               while (data[idx][0] == prev_state){

                if (data[idx][curr_column] != '--') selected_total += parseFloat(data[idx][curr_column]);
                idx++;
                if (idx == 219) break;
              }
              ptr_selected[state_abbre_to_state[prev_state]] = selected_total;
              // console.log(prev_state)
              corr_selected.set(state_abbre_to_state[prev_state],selected_total)
              console.log(ptr_selected)
            }
          }else{
            console.log(data)
            let prev_state = 'CT';
            let idx = 2;
            while (idx < data.length - 1){
             prev_state = data[idx][0];
              while (data[idx][0] == prev_state){
                let ground_columns = [10,20,27,45,54,64,69,76,141];
                let i = 0;
                while(i in ground_columns){
                  let curr = ground_columns[i];
                  
                  if (data[idx][curr] != '--') ground += parseFloat(data[idx][curr]);
                  else{
                    ground += 0;
                  }
                  i++;
                }

                corr_ground.set(state_abbre_to_state[data[idx][0]],ground)      
                ptr_ground[state_abbre_to_state[data[idx][0]]] = ground
        
                // surface = parseFloat(data[idx][13]) + parseFloat(data[idx][21]) + parseFloat(data[idx][30]) + parseFloat(data[idx][46]) + parseFloat(data[idx][55]) + parseFloat(data[idx][65]) + parseFloat(data[idx][72])+parseFloat(data[idx][79])+parseFloat(data[idx][142]);
                let surface_columns = [13,21,30,46,55,65,72,79,142];
                i = 0;
                while(i in surface_columns){
                  let curr = surface_columns[i];
                  if (data[idx][curr] != '--')  surface += parseFloat(data[idx][curr]);
                  else{
                    surface += 0;
                  }
                  i++;
                }
                corr_surface.set(state_abbre_to_state[data[idx][0]],surface);
                ptr_surface[state_abbre_to_state[data[idx][0]]] = surface;
        
                total = ground + surface;
                corr_total.set(state_abbre_to_state[data[idx][0]],total)
                ptr_total[state_abbre_to_state[data[idx][0]]] = total

                idx++;

                if (idx == 219) break;
            }
            ground = 0;
            surface = 0;
            total = 0;
          }

          }

        }else{
          if (selected_category != null){
            let curr_column = selected_category[1];
            let newD = [];
            for (let i = 2; i < data.length; i++){
              console.log(state_to_state_abre[selected_state]);
              if (data[i][0] == state_to_state_abre[selected_state]){
                newD.push(data[i]);
              }
            }
            data_ready=newD;
            console.log(data_ready)

            let idx = 0;
            let selected_total = 0;
            while(idx < data_ready.length){
              if (data_ready[idx][curr_column] != '--')  selected_total = parseFloat(data_ready[idx][curr_column]);
              ptr_selected[data_ready[idx][4]] = selected_total;
              // console.log(prev_state)
              corr_selected.set(data_ready[idx][4],selected_total)
              idx++;
            }

          }
          console.log(ptr_selected)

                              // get data for that particular state
                              let newD = [];
            for (let i = 2; i < data.length; i++){
              console.log(state_to_state_abre[selected_state]);
              if (data[i][0] == state_to_state_abre[selected_state]){
                newD.push(data[i]);
              }
            }
            data_ready=newD;
            console.log(data_ready)

            let idx = 0;
        while (idx < data_ready.length){

                let ground_columns = [10,20,27,45,54,64,69,76,141];
                let i = 0;
                while(i in ground_columns){
                  let curr = ground_columns[i];
                  
                  if (data_ready[idx][curr] != '--') ground += parseFloat(data_ready[idx][curr]);
                  else{
                    ground += 0;
                  }
                  i++;
                }

                corr_ground.set(data_ready[idx][4],ground)      
                ptr_ground[data_ready[idx][4]] = ground
        
                // surface = parseFloat(data[idx][13]) + parseFloat(data[idx][21]) + parseFloat(data[idx][30]) + parseFloat(data[idx][46]) + parseFloat(data[idx][55]) + parseFloat(data[idx][65]) + parseFloat(data[idx][72])+parseFloat(data[idx][79])+parseFloat(data[idx][142]);
                let surface_columns = [13,21,30,46,55,65,72,79,142];
                i = 0;
                while(i in surface_columns){
                  let curr = surface_columns[i];
                  if (data_ready[idx][curr] != '--')  surface += parseFloat(data_ready[idx][curr]);
                  else{
                    surface += 0;
                  }
                  i++;
                }
                corr_surface.set(data_ready[idx][4],surface);
                ptr_surface[data_ready[idx][4]] = surface;
        
                total = ground + surface;
                corr_total.set(data_ready[idx][4],total)
                ptr_total[data_ready[idx][4]] = total

                idx++;

                ground = 0;
                surface = 0;
                total = 0;

                // if (idx == 219) break;
            }
            console.log(ptr_ground)


      


          }

        
              console.log(ptr_ground)
              console.log(ptr_surface)
              console.log(ptr_total)
      }
  
  
      // let ptr_selected = {};
      // let corr_selected = new Map();
  
      const getData_for_selected = () => {
        if (selected_category == null){
          return;
        }

        let curr_column = selected_category[1];
        console.log(selected_category)
        if(selected_state == null){
          // console.log(data)
      
          let prev_state = 'CT';
          let idx = 2;
          while (idx < data.length - 1){
            prev_state = data[idx][0];
            let selected_total = 0;
              while (data[idx][0] == prev_state){

                if (data[idx][curr_column] != '--') selected_total += parseFloat(data[idx][curr_column]);
                idx++;
                if (idx == 219) break;
              }
              ptr_selected[state_abbre_to_state[prev_state]] = selected_total;
              // console.log(prev_state)
              corr_selected.set(state_abbre_to_state[prev_state],selected_total)
          }
          console.log(ptr_selected)


        }else{
            let newD = [];
            for (let i = 2; i < data.length; i++){
              console.log(state_to_state_abre[selected_state]);
              if (data[i][0] == state_to_state_abre[selected_state]){
                newD.push(data[i]);
              }
            }
            data_ready=newD;
            console.log(data_ready)

            let idx = 0;
            let selected_total = 0;
            while(idx < data_ready.length){
              if (data_ready[idx][curr_column] != '--')  selected_total = parseFloat(data_ready[idx][curr_column]);
              ptr_selected[data_ready[idx][4]] = selected_total;
              // console.log(prev_state)
              corr_selected.set(data_ready[idx][4],selected_total)
              idx++;
            }
            
            

        }

        console.log(ptr_selected)
      }


        // for (let idx = 0; idx < data.length; idx++) {
        //   ptr_selected[data[idx][4]] = parseFloat(data[idx][curr_column]);
        // }
  
        
  
      
  
    // const getData = () => {
  
    //   // console.log(typeof data[1]);
  
    //   // ground columns = [10,20,27,45,54,64,69,76,141];
    //   // surface columns = [13,21,30,46,55,65,72,79,142]
    //   if (type == 'ground' || type == null){
        
        
    //   }
    //   return 
  
    // }
    const create_initial = () => {
      
      if (layer) {
        layer.remove();
      }
  
      let vals = {};
  
  
  
      
      layer = L.geoJSON(geojson, {
        style: (feature) => {
          
          const fips = selected_state == null ? feature.properties.name : feature.properties['GEOID'];

          const idx_ground = ptr_ground[fips];
          const idx_total= ptr_total[fips];
          const val = (idx_ground/ idx_total);
          // console.log(val)
          vals[fips] = val;
          // console.log(val)
          let clr = interpolateYlOrRd(val*2);
          let s = {color:'grey',fillColor: clr, fillOpacity: 0.7};
          return s;
        },
          onEachFeature(feature, layer){
          layer.on('mouseover', function (){
            this.setStyle({
              'fillOpacity': 1.5
            });
            let mouseovered = selected_state == null ? feature.properties.name : feature.properties['GEOID'];

            // let number = corr.get(geoid);
            let message = "Percent Ground Water:" + Math.round(vals[mouseovered] * 100) + "%";
            this.bindPopup(message).openPopup();
          });
          layer.on('mouseout', function(){
            this.setStyle({
              'fillOpacity': 0.7
            });
          });
          layer.on('click', function(){
            if(selected_state) return;
            let state_name = this.feature.properties['name'];
            selected_state = state_name;
            layer.remove();
            update_geojson();
          })
        }
      })
        .addTo(container);
        // console.log(222);
  
    }
    
  
      const updateLayer = () => {
  
        // if (selected === null){
        //   create_initial();
        //   return;
        // }
  
        if (layer) {
        layer.remove();
      }
      console.log(ptr_ground)
  
      let columns;
      let column_names;
      let color;

      let vals = {};
      // console.log(ptr_ground)
  
      
      layer = L.geoJSON(geojson, {
        style: (feature) => { 
          let extent_data = [];
  
  
          const fips = selected_state == null ? feature.properties.name : feature.properties['GEOID'];
          let idx_num;
          let idx_total;
          if (type == 'ground' || type == null){
            idx_num = selected_category == null ? ptr_ground[fips] : ptr_selected[fips];
            idx_total = selected_category == null ? ptr_total[fips] : ptr_ground[fips];
          }else{
            idx_num = selected_category == null ? ptr_surface[fips] : ptr_selected[fips];
            idx_total = selected_category == null ? ptr_total[fips] : ptr_surface[fips];
          }
          console.log(idx_num)
          // const idx_ground = ptr_ground[fips];
          // const idx_total= ptr_total[fips];
          console.log(idx_total)
          const val = (idx_num/ idx_total);
  
  
          vals[fips] = val;
          // console.log(vals);
          let clr;
          if (type =='ground'){
            clr = interpolateYlOrRd(val);
          }else{
            clr = interpolateGnBu(val);
          }
          
          // console.log(clr)
          let s = {color:'grey',fillColor: clr, fillOpacity: 0.7};
          
          return s;
        },
        onEachFeature(feature, layer){
          layer.on('mouseover', function (){
            this.setStyle({
              'fillOpacity': 1.5
            });
            let mouseovered = selected_state == null ? feature.properties.name : feature.properties['GEOID'];
            // let number = corr.get(geoid);
            let message;
            if (selected_category){
              message = "Percent " + selected_category[0] + ' in' + type + ' Water' + ' : ' + Math.round(vals[mouseovered] * 100) + "%";

            }else{
              message = "Percent " + type + ' Water' + ' : ' + Math.round(vals[mouseovered] * 100) + "%";

            }
            this.bindPopup(message).openPopup();
          });
          layer.on('mouseout', function(){
            this.setStyle({
              'fillOpacity': 0.7
            });
          });
          layer.on('click', function(){
            if(selected_state) {
              selected_county = this.feature.properties['GEOID']
              console.log(selected_county)
              return;
            };
            let state_name = this.feature.properties['name'];
            selected_state = state_name;
            layer.remove();
            update_geojson();
          })
        }
      })
        .addTo(container);
      }
  
      const updateSelected =() => {
        if (selected_category == null ){
          updateLayer();
          return;
        }
        // console.log('updating selected!')
        let vals = {};
      // console.log(ptr_ground)
  
      
      layer = L.geoJSON(geojson, {
        style: (feature) => { 
          let extent_data = [];
  
  
          const fips = selected_state == null ? feature.properties.name : feature.properties['GEOID'];
          let idx_total;
          if (type == 'ground' || type == null){
            idx_total = ptr_ground[fips];
          }else{
            idx_total = ptr_surface[fips];
          }
          // console.log(idx_total)

          const val = ptr_selected[fips] / idx_total;
          // const idx_ground = ptr_ground[fips];
          console.log(val)
  
          vals[fips] = val;
          // console.log(vals);
          let clr;
          if (type =='ground'){
            clr = interpolateYlOrRd(val);
          }else{
            clr = interpolateGnBu(val);
          }
          let s = {color:'grey',fillColor: clr, fillOpacity: 0.7};
          return s;
        },
        onEachFeature(feature, layer){
          layer.on('mouseover', function (){
            this.setStyle({
              'fillOpacity': 1.5
            });
            let state_name = this.feature.properties['name'];
            // let number = corr.get(geoid);
            let message = "Percent " + selected_category[0] + ' in ' + type + ' : ' + Math.round(vals[state_name] * 100) + "%";
            this.bindPopup(message).openPopup();
          });
          layer.on('mouseout', function(){
            this.setStyle({
              'fillOpacity': 0.7
            });
          });
        }
      })
        .addTo(container);
      }


      async function backToRegional(){
        selected_state = null;
        get_geojson.update(n => "states.json")
      }
  
  
        
      // }
  
    $: {
      console.log(data);
      getData();
      create_initial();
      // create_initial();
    }

    $:{
      console.log(geojson);
      getData();
      updateLayer();
    }
  
    // $: {
    //   console.log(selected_category)
    //   getData_for_selected();
    //   updateSelected();
    // }
  
    $: {
      console.log(type)
      console.log(selected_category)
      getData();
      // if (selected_category){
      //   getData_for_selected()
      // }
      updateLayer();
    }
  
  
  </script>
  
  
  <style>
  
  </style>
  <div class = "counties">
    <p>{get_geojson}</p>
    <!-- {#if selected_state == null}
      <button on:click={backToRegional}>Back to Regional View</button>
    {/if} -->
    <!-- {#if selected_state != null}
        <CountyData
            geojson={data}
              bind:selected_category={selected_category} 
              data={data}
              bind:type={type}
              bind:selected_state={selected_state}
        >
        </CountyData>
    {/if} -->
  </div>
  
  
  <slot />
  