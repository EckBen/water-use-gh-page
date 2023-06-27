<script lang="ts">
    // import { arc, pie } from "d3-shape";
    import {onMount} from 'svelte';
    // import { colorCategoricalDragon } from "$utils/brand";
    import {schemeRdBu, interpolateSpectral, interpolateYlOrRd, interpolateGnBu, schemeSet1, schemeSet3, schemeTableau10} from 'd3-scale-chromatic';
    import * as d3 from 'd3';
    import CountyData from './CountyData.svelte';
    import { arc } from 'd3';
    import { update_await_block_branch } from 'svelte/internal';
    export let data: Array<Array<string>>;
    export let selected_category: [string,number] = null;
    export let type: string = null;
    export let selected_state: string = null;
    export let selected_county: string = null;
    // export let group:[string] = null;

    // export let data: Array<Array<string>>;
    // export let selected: [string,number] = null;
    // export let type: String = null;

    // [2, 3, 5, 8, 13, 21]

    // onMount(() => {
    // 	console.log(data)
  	// });

    const width = 1000,
    height = 450,
    margin = {left: 40, right: 100};
    var radius = Math.min(width, height) / 2 - margin.left;
    var innerRadius = 0;
    let labelHeight = 18;


    let columns;
    let column_names;
    let pData = new Object();
    let color;

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
    // let reload = 0;

    // Use                                     Ground Water            Surface Water

// Public Supply                           K  11                             N 14
// Domestic Supply                         U  21                             V 22
// Industrial                                      AB      28                        AE 31
// Crop Irrigation                         AT      46                        AU 47
// Golf Course Irrigation          BC  55                            BD 56
// Livestock                                       BL   65                           BM 66
// Aquaculture                             BQ      70                        BT 73
// Mining                                  BX      77                        CA 80
// Thermoelectric              CG + DK   143                CJ + DN  144


    const getData = () => {
      // console.log(data)
      if (type=="ground" || type == null){
        console.log(type + 'data')
          columns = [10,20,27,45,54,64,69,76,141];
          column_names = ["Public Supply","Domestic Supply","Industrial","Crop Irrigatin","Golf Course Irrigation","LiveStock","Aquaculture","Mining","thermoelectric"];
          // color = d3.scaleOrdinal().domain(column_names).range(['yellow','blue','palevioletred','plum','purple','rebeccapurple','thistle','black','red']);
          color = d3.scaleOrdinal().domain(column_names).range(schemeTableau10);
      }else{
        console.log(type + 'data')
          columns = [13,21,30,46,55,65,72,79,142];
          column_names = ["Public Supply","Domestic Supply","Industrial","Crop Irrigatin","Golf Course Irrigation","LiveStock","Aquaculture","Mining","thermoelectric"];
          // color = d3.scaleOrdinal().domain(column_names).range(['pink','peachpuff','palevioletred','plum','purple','rebeccapurple','thistle','black','red']);
          color = d3.scaleOrdinal().domain(column_names).range(schemeTableau10);
      }
      pData = new Object();

      if (selected_state == null){
        for (let idx in columns){
            let tot = 0;
            for (let i = 2; i < data.length; i++){
              // console.log(data[i][columns[idx]])
              if (data[i][columns[idx]] != '-'){
                tot += Number(data[i][columns[idx]]);
              }
                
            }
            pData[column_names[idx]] = tot;
            // clr = interpolateSpectral(tot*10);
        }
      }else{
        if (!selected_county){
          let data_ready = [];
          for (let i = 2; i < data.length; i++){
                if (data[i][0] == state_to_state_abre[selected_state]){
                  data_ready.push(data[i]);
                }
          }

          for (let idx in columns){
              let tot = 0;
              for (let i = 2; i < data_ready.length; i++){
                // console.log(data[i][columns[idx]])
                if (data_ready[i][columns[idx]] != '-'){
                  tot += Number(data_ready[i][columns[idx]]);
                }
                  
              }
              pData[column_names[idx]] = tot;
              // clr = interpolateSpectral(tot*10);
          }

        }else{
          console.log('here')
          let data_ready;
          for (let i = 2; i < data.length; i++){
                if (data[i][4] == selected_county){
                  data_ready = data[i];
                }
          }
          console.log(data_ready)

          for (let idx in columns){
            // if (data_ready[idx][columns[idx]]){
              pData[column_names[idx]] = data_ready[columns[idx]]
            // }else{
            //   pData[column_names[idx]] = 0;
            // }
              // pData[column_names[idx]] = data_ready[idx][columns[idx]];
              // clr = interpolateSpectral(tot*10);
          }

        }

      }

        
        
      console.log(pData)
      return pData;
    }

    // let path;
    // let pie;
    // let pieData;
    
    // const drawChart =() => {
      // let arcTween;
      var svg;
      // var local = d3.local();

      const drawSVG = () => {
        svg = d3.select("#mydatavis")
          .append("svg")
            .attr("width", width)
            .attr("height", height)
          .append("g")
          .attr('id','pieChart')
            .attr("transform", `translate(${width / 2}, ${height / 2})`);
        var g = svg.append('g')
          .attr('transform', 'translate(200, 75)');
        svg.append("g")
	        .attr("class", "labels");
        svg.append("g")
          .attr("class", "lines");

      }
      
      // pie = d3.pie()
      //     .value(function(d) { return d[1] })
      // pieData = pie(Object.entries(pData));
      // console.log('drawing chart')

      const update = () =>  {
        console.log(type);
        let curr_data = getData();
        console.log(curr_data)
        var key = function(d){ return d.data[0]; };
        var num = function(d){return d.data[1]}
        
        var pie = d3.pie()
          .value(function(d) { return d[1] })
        var pieData = pie(Object.entries(curr_data));
        console.log(pieData)
        var arcGen = d3.arc()
          .innerRadius(0)
          .outerRadius(radius);
        var outerArc = d3.arc()
          .innerRadius(radius * 0.9)
          .outerRadius(radius * 0.9);
        console.log(pieData)
        var path = svg.selectAll("path")
                      .data(pieData,key)

        path
            .enter()
            .append('path')
            .attr('id', function(d){ return key(d)})
            .on('click', handleClick)
            .on('mouseover',function(e){

              console.log(e)
            })
            .merge(path)
            .transition()
            .duration(300)
            .attr('fill', function(d){ return(color(d.data[0])) })
            .attr('d',arcGen)
        // var path = svg.selectAll("g.arc")
        //               .data(pieData,key)
        //               .enter()
        //               .append('g')
        //               .attr('class','arc')
        //               .attr('tranform',"translate(" + radius + "," + radius + ")")
        //               .on('mouseover',function(d){
        //                 d3.select("#tooltip")
        //                 .style("left", d3.event.pageX + "px")
        //                 .style("top", d3.event.pageY + "px")
        //                 .style("opacity", 1)
        //                 .select("#value")
        //                 .text('hi');
        //               })
        

        // path
        //     .append('path')
        //     .attr('id', function(d){ return key(d) + ',' + num(d)})
        //     .on('click', handleClick)
        //     .on('mouseover',function(e){

        //       console.log(e)
        //     })
        //     .merge(path)
        //     .transition()
        //     .duration(300)
        //     .attr('fill', function(d){ return(color(d.data[0])) })
        //     .attr('d',arcGen)

        
        path.exit().remove()


      // legend dimensions
        var legendRectSize = 15; // defines the size of the colored squares in legend
        var legendSpacing = 6; // defines spacing between squares
        var legend = svg.selectAll('.legend') // selecting elements with class 'legend'
          .data(color.domain()) // refers to an array of labels from our dataset
          .enter() // creates placeholder
          .append('g') // replace placeholders with g elements
          .attr('class', 'legend') // each g is given a legend class
          .attr('transform', function(d, i) {                   
              var height = legendRectSize + legendSpacing; // height of element is the height of the colored square plus the spacing      
              var offset =  height * color.domain().length / 2; // vertical offset of the entire legend = height of a single element & half the total number of elements  
              var horz = 14 * legendRectSize; // the legend is shifted to the left to make room for the text
              var vert = i * height - offset; // the top of the element is hifted up or down from the center using the offset defiend earlier and the index of the current element 'i'               
                return 'translate(' + horz + ',' + vert + ')'; //return translation       
          });

      // adding colored squares to legend
        legend.append('rect') // append rectangle squares to legend                                   
          .attr('width', legendRectSize) // width of rect size is defined above                        
          .attr('height', legendRectSize) // height of rect size is defined above                      
          .style('fill', color) // each fill is passed a color
          .style('stroke', color) // each stroke is passed a color
          .on('click', function() {
              // enabled property: https://codepen.io/thecraftycoderpdx/pen/jZyzKo
        
        });

        // let rects = legend.selectAll('rect').data(pieData)
        // rects
        // .enter()
        // .append('rect')
        // .attr('width', legendRectSize) // width of rect size is defined above                        
        //   .attr('height', legendRectSize) // height of rect size is defined above                      
        //   .attr('fill', (d) => {
        //     console.log('filled'+ d.data[0] + color(d.data[0]))
        //     return color(d.data[0])
        //   })
        //   .style('stroke', d => color(d.data[0])) // each stroke is passed a color


        // rects.call(update => update.transition()
        // .attr('fill', d => color(d.data[0])))
        // .attr('fill', d => color(d.data[0]))
        // .join(
        //   enter => enter.append('rect')                                
        //   .attr('width', legendRectSize) // width of rect size is defined above                        
        //   .attr('height', legendRectSize) // height of rect size is defined above                      
        //   .attr('fill', (d) => {
        //     console.log('filled'+ d.data[0] + color(d.data[0]))
        //     return color(d.data[0])
        //   })
        //   .style('stroke', d => color(d.data[0])) // each stroke is passed a color
        //   .call(enter => enter.transition().attr('opacity',1)),
        //   update => update.call(update => update.transition()
        //   .attr('fill', d => color(d.data[0])))
        //   ,
        // );

    // adding text to legend
      legend.append('text')                                    
          .attr('x', legendRectSize + legendSpacing)
          .attr('y', legendRectSize - legendSpacing)
          .text(function(d) { return d; }); // return label
                  
        }

        


    let handleClick = (e) => {

      let getIndex = (this_col_name) =>{
        for (let i in column_names){
          if (column_names[i] == this_col_name){
            // console.log(column_names[i])
            return columns[Number(i)];
          }
        }
      }

      selected_category = [e.target.id,getIndex(e.target.id)];

      
      

    }


    

    $: {
      console.log(data)
      console.log('data changed')
        drawSVG();
        update();

    }

    $: {
      console.log(type)

        // getData()
        update()

      // console.log(type)
      // getData()
      // drawChart()
      // reload++;
    }

    $:{
      console.log(selected_state);
      update();
    }
    $:{
      console.log(selected_county);
      update();
    }
  </script>

  <!-- <h3>Check out the breakdown for {type} water usage!</h3> -->
  <div  id="mydatavis">
    <!-- <svg></svg> -->
  </div>

  <div id="tooltip" class="hidden">

  </div>
  
  <!-- <Plot the svg paths  -->

  <style>
    .type{
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
  </style>
  


  