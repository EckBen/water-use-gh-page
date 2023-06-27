<script lang="ts">
  import * as L from 'leaflet';
  import {schemeRdBu, interpolateSpectral} from 'd3-scale-chromatic';
  import {
    getContext,
    setContext,
    onDestroy,
    createEventDispatcher,
    onMount,
  } from 'svelte';
  import * as d3 from 'd3';
  import { feature } from 'topojson-client';

  const dispatch = createEventDispatcher();

  export let geojson: any;
  export let selected_category: [string,number] = null;
  export let data: Array<Array<string>> = null;
  export let type: string = null;
  export let selected_state: string = null;

  const container = getContext<() => L.LayerGroup>('layerGroup')();
  
  let layer: L.GeoJSON = null;
  // const create_initial = () => {

  //   layer=L.geoJSON(geojson, {
  //     style: (feature) => {
  //       let s = {fillColor: "blue"};
  //       return s;
  //     }
  //   })
  //     .on('mouseover', e => function(){
  //       layer.setStyle({fillOpacity: 3})
  //     })
  //     .on('mouseout', (e) => dispatch('mouseout', e))
  //     .on('click', (e) => dispatch('click', e))
  //     // .addTo(container);
  //     // selected = ['default',100]
  // }
  // selected = ['default',100]
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
    let corr_ground = new Map();
    let corr_surface = new Map();
    let corr_total = new Map();
    let ground = 0;
    let surface = 0;
    let total = 0;
    let ptr_ground = {}
    let ptr_surface = {}
    let ptr_total = {}
    let topoData;

    async function getTopoData(){
      const response = await fetch(geojson);
      const json = await response.json();
      // console.log(json);
      topoData = feature(json, json.objects.cb_2015_connecticut_county_20m);
      console.log(geojson)
      console.log(topoData)
    }

    

    const getData = () => {
      // get data for that particular state
      let newD = [];
      for (let i = 2; i < data.length; i++){
        console.log(state_to_state_abre[selected_state]);
        if (data[i][0] == state_to_state_abre[selected_state]){
          newD.push(data[i]);
        }
      }
      data=newD;
      // console.log(newD)
      for (let idx = 0; idx < data.length; idx++) {
        // console.log(parseFloat(data[idx][7]) + parseFloat(data[idx][20]) + parseFloat(data[idx][29]) + parseFloat(data[idx][36]) + parseFloat(data[idx][63]) + parseFloat(data[idx][68]) + parseFloat(data[idx][77]));
        // console.log(222);
        if (type == 'ground' || type == null){
          
        }
        ground = parseFloat(data[idx][10]) + parseFloat(data[idx][20]) + parseFloat(data[idx][27]) + parseFloat(data[idx][45]) + parseFloat(data[idx][54]) + parseFloat(data[idx][64]) + parseFloat(data[idx][69])+parseFloat(data[idx][76])+parseFloat(data[idx][141]);
        corr_ground.set(data[idx][4],ground)      
        ptr_ground[data[idx][4]] = ground

        surface = parseFloat(data[idx][13]) + parseFloat(data[idx][21]) + parseFloat(data[idx][30]) + parseFloat(data[idx][46]) + parseFloat(data[idx][55]) + parseFloat(data[idx][65]) + parseFloat(data[idx][72])+parseFloat(data[idx][79])+parseFloat(data[idx][142]);
        corr_surface.set(data[idx][4],surface);
        ptr_surface[data[idx][4]] = surface;

        total = ground + surface;
        corr_total.set(data[idx][4],total)
        ptr_total[data[idx][4]] = total
      }

      console.log(ptr_ground)
      console.log(ptr_surface)
      console.log(ptr_total)

    }


    let ptr_selected = {};
    let corr_selected = new Map();

    const getData_for_selected = () => {
      if (selected_category == null){
        return;
      }
      let curr_column = selected_category[1];
      for (let idx = 0; idx < data.length; idx++) {
        ptr_selected[data[idx][4]] = parseFloat(data[idx][curr_column]);
      }

      console.log(ptr_selected)

    }

  // const getData = () => {

  //   // console.log(typeof data[1]);

  //   // ground columns = [10,20,27,45,54,64,69,76,141];
  //   // surface columns = [13,21,30,46,55,65,72,79,142]
  //   if (type == 'ground' || type == null){
      
      
  //   }
  //   return 

  // }
  const create_initial = () => {
    console.log('removed')
    
    if (layer) {
      console.log('removed')
      layer.remove();
    }

    let vals = {};

    console.log(topoData)

    
    layer = L.geoJSON(topoData, {
      style: (feature) => {
        const fips = feature.properties['GEOID'];
        const idx_ground = ptr_ground[fips];
        const idx_total= ptr_total[fips];
        const val = (idx_ground/ idx_total);
        vals[fips] = val;
        let clr = interpolateSpectral(val*2);
        let s = {color:'grey',fillColor: clr, fillOpacity: 0.7};
        return s;
      },
        onEachFeature(feature, layer){
        layer.on('mouseover', function (){
          this.setStyle({
            'fillOpacity': 1.5
          });
          let geoid = this.feature.properties['GEOID'];
          // let number = corr.get(geoid);
          let message = "Percent Ground Water:" + Math.round(vals[geoid] * 100) + "%";
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

    let columns;
    let column_names;
    let color;
    
    // if (selected[2] == "usage"){
    //       columns = [18,26,35,47,56,65,74,83,97];
    //       column_names = ["Public Supply","Self-supplied domestic","Industrial","Crop Irrigatin","Golf Course Irrigation","LiveStock","Aquaculture","Mining","thermoelectric consumptive"];
    //       let colors = ['pink','peachpuff','palevioletred','plum','purple','rebeccapurple','	thistle'];
    //       // color = d3.scaleSequential().domain([0,0.7]).interpolator(d3.interpolate('beige','pink'));
    //       color = d3.scaleOrdinal().domain(column_names).range(colors)

    //     }else{
    //       columns = [7,20,29,36,63,68,77];
    //       let abbrev_to_use = new Map([
    //         ['AQ-WGWTo' , 'Public Supply Ground Water Withdrawal'],
    //         ['DO-WGWFr' , 'Domestic Self-supplied Ground Water Withdrawal'],
    //         ['IN-WGWTo' , 'Industrial Self-supplied Ground Water Withdrawal'],
    //         ['IR-WGWFr' , 'Irrigation Ground Water Withdrawal'],
    //         ['LI-WGWFr' , 'Livestock Ground Water Withdrawal'],
    //         ['MI-WGWTo' , 'Aquaculture Ground Water Withdrawal'],
    //         ['PS-GWPop' , 'Mining Ground Water Withdrawal']
    //       ]);

    //       column_names = ["Public Supply Ground Water Withdrawal","Domestic Self-supplied Ground Water Withdrawal","Industrial Self-supplied Ground Water Withdrawal","Irrigation Ground Water Withdrawal","Golf Course Irrigation","Livestock Ground Water Withdrawal","Aquaculture Ground Water Withdrawal","Mining Ground Water Withdrawal"];
    //       let colors = ['steelblue','aqua','dodgerblue','cadetblue','cornflowerblue','cyan','lavender']
    //       color = d3.scaleOrdinal().domain(column_names).range(colors)
    //     }

    // let corr_ground = new Map();
    // let corr_total = new Map();
    // let ground = 0;
    // let total = 0;
    // let ptr_ground = {}
    // let ptr_total = {}
    // // console.log(typeof data[1]);
    // for (let idx = 1; idx < data.length; idx++) {

    //   ground = parseFloat(data[idx][selected[1]]);
    //   corr_ground.set(data[idx][4],ground)
    //   ptr_ground[data[idx][4]] = ground
    //   total = parseFloat(data[idx][18]) + parseFloat(data[idx][26]) + parseFloat(data[idx][35]) + parseFloat(data[idx][47]) + parseFloat(data[idx][56]) + parseFloat(data[idx][65]) + parseFloat(data[idx][74]) + parseFloat(data[idx][83]) + parseFloat(data[idx][97]);
    //   // [18,26,35,47,56,65,74,83,97]
    //   corr_total.set(data[idx][4],total)
    //   ptr_total[data[idx][4]] = total
    // }
    let vals = {};
    // console.log(ptr_ground)

    
    layer = L.geoJSON(topoData, {
      style: (feature) => { 
        let extent_data = [];


        const fips = feature.properties['GEOID'];
        let idx_num;
        if (type == 'ground' || type == null){
          idx_num = ptr_ground[fips];
        }else{
          idx_num = ptr_surface[fips];
        }
        // const idx_ground = ptr_ground[fips];
        const idx_total= ptr_total[fips];
        const val = (idx_num/ idx_total);


        vals[fips] = val;
        // console.log(vals);
        let clr = interpolateSpectral(val*2);
        let s = {color:'grey',fillColor: clr, fillOpacity: 0.7};
        return s;
      },
      onEachFeature(feature, layer){
        layer.on('mouseover', function (){
          this.setStyle({
            'fillOpacity': 1.5
          });
          let geoid = this.feature.properties['GEOID'];
          // let number = corr.get(geoid);
          let message = "Percent " + type + ' Water' + ' : ' + Math.round(vals[geoid] * 100) + "%";
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

    const updateSelected =() => {
      if (selected_category == null ){
        updateLayer();
        return;
      }
      // console.log('updating selected!')
      let vals = {};
    // console.log(ptr_ground)

    
    layer = L.geoJSON(topoData, {
      style: (feature) => { 
        let extent_data = [];


        const fips = feature.properties['GEOID'];
        let idx_total;
        if (type == 'ground' || type == null){
          idx_total = ptr_ground[fips];
        }else{
          idx_total = ptr_surface[fips];
        }
        const val = ptr_selected[fips] / idx_total;
        // const idx_ground = ptr_ground[fips];


        vals[fips] = val;
        // console.log(vals);
        let clr = interpolateSpectral(val*2);
        let s = {color:'grey',fillColor: clr, fillOpacity: 0.7};
        return s;
      },
      onEachFeature(feature, layer){
        layer.on('mouseover', function (){
          this.setStyle({
            'fillOpacity': 1.5
          });
          let geoid = this.feature.properties['GEOID'];
          // let number = corr.get(geoid);
          let message = "Percent " + selected_category[0] + ' in ' + type + ' : ' + Math.round(vals[geoid] * 100) + "%";
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


      
    // }

  $: {
    if (data){
      getData();
      
    }

    if (topoData){
      getTopoData();
      
    }
    create_initial();
    // console.log(data);
    // // console.log(topoData);
    // console.log(selected_state)
    // // getTopoData();
    // getData();
    // // // console.log(topoData)
    // create_initial();
    // create_initial();
  }

  // $:{
  //   // console.log(topoData)
  //   getTopoData();
  //   getData();
  //   create_initial();
  // }

  // $: {
  //   console.log(topoData)
  //   getTopoData();
  // }

  $: {
    console.log(selected_category)
    getData_for_selected();
    updateSelected();
  }

  $: {
    console.log(type)
    updateLayer();
  }


</script>


<style>

</style>


<slot />
