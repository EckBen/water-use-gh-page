<script lang="ts">
	import {onMount} from 'svelte';
	import * as d3 from "d3";
    import { easeCircle } from 'd3';

	export let data: Array<Array<string>> = null;


	onMount(() => {
    	console.log(data)
  	});

	let height = 600;
	let width = 600;
	let innerHeight;
	let innerWidth;
	let margin;
	let ptr_ground;
	let yScale;
	let xScale;
	const updateGraph = () => {
    	let ground = 0;
    	ptr_ground = [];
		let cols = [7,20,29,36,63,68,77];
		let cols_names = [];
		for (let idx = 1; idx < data.length; idx++) {
			let ground_this = {};
			let county = data[idx][2].split(' ');
			ground_this['county'] = county[0];

			// get each source
			for (let i = 0; i < cols.length; i++){
				let this_col = data[0][cols[i]];
				if (idx == 1){
					cols_names.push(this_col);
				}
				
				ground_this[this_col] = parseFloat(data[idx][cols[i]]);
			}
			ground = parseFloat(data[idx][7]) + parseFloat(data[idx][20]) + parseFloat(data[idx][29]) + parseFloat(data[idx][36]) + parseFloat(data[idx][63]) + parseFloat(data[idx][68]) + parseFloat(data[idx][77]);
      		// corr_ground.set(data[idx][4],ground)

			ground_this['use'] = ground;
      		// ptr_ground[data[idx][4]] = ground
			// console.log(ptr_ground)
			ptr_ground.push(ground_this)
		}

		console.log(ptr_ground);

		const exp_data = [
			{county: 'A', Ground:40, Underground:60, total:100},
			{county: 'B', Ground:28, Underground:72, total: 100},
			{county: 'C', Ground:10, Underground:90, total: 100}
		]

		const bar = d3.select("#bar");
                    const width_bar = height;
                    const height_bar = width;
                    const margins_bar = { top: 100, right: 30, bottom: 40, left: 40 };
                    let chartWidth_bar = width_bar - margins_bar.left - margins_bar.right;
                    const chartHeight_bar = height_bar - margins_bar.top - margins_bar.bottom;
                    let annotations_bar = bar.append("g").attr("id", "annotations");
                    let chartArea_bar = bar.append('g')
                        .attr('transform', `translate(${margins_bar.left},${margins_bar.top})`);

		const xScale = d3.scaleBand()
                            .domain(cols_names)
                            .range([0, chartWidth_bar])
                            .padding(0.05);

							// const yExtent = d3.extent()
                        const yScale = d3.scaleLinear()
                            .domain([0,100])
                            .range([chartHeight_bar, 0]);

                        console.log(cols_names);


                        let leftAxis = d3.axisLeft(yScale);
                        bar.append('g')
                            .attr('class', 'axis')
                            .attr('transform', `translate(${margins_bar.left - 10},${margins_bar.top})`)
                            .call(leftAxis);

                        let leftGridlines = d3.axisLeft(yScale)
                            .tickSize(-chartWidth_bar - 10)
                            // .tickFormat('');

                        bar.append('g')
                            .attr('class', 'gridlines')
                            .attr('transform', `translate(${margins_bar.left - 10},${margins_bar.top})`)
                            .call(leftGridlines);


                        chartArea_bar.selectAll('rect.bar').data( exp_data )
                                .join('rect').attr('class','bar_private')
                                .attr("fill", 'steelblue')
                                .attr("x", d => xScale(d.county))
                                .attr("y", d => yScale(d['ground']))
                                .attr("height", d => yScale(0) - yScale(d['ground']))
                                .attr("width", xScale.bandwidth());

                        chartArea_bar.selectAll('rect.bar').data( exp_data )
                                .join('rect').attr('class','bar_dpw')
                                .attr("fill", 'orange')
                                .attr("x", d => xScale(d.county))
                                .attr("y", d => yScale(d.total))
                                .attr("height", d => yScale(d['ground']) - yScale(d.total))
                                .attr("width", xScale.bandwidth())
                                .attr("text",d => d.county)
                        
                        // chartArea_bar.selectAll('text').data( total_count )
                        //         .join('text')
                        //         .attr('class', 'labels')
                        //         .text(d => d.qspecies)
                        //         .attr('x', 0)
                        //         .attr("y", 0)
                        //         .attr('transform', d => `translate(${xScale(d.qspecies) + xScale.bandwidth()/2},${yScale(d.total) - 10}) rotate(-60)`)

                    
                    // requestData_bar();
		

		// height = 600;
		// width = 840;
		// margin = { top: 20, bottom: 20, left: 100, right: 20 };
		// innerHeight = height - margin.top - margin.bottom;
  		// innerWidth = width - margin.left - margin.right;

		// yScale = d3.scaleBand()
		// 	.domain(ptr_ground.map((d) => d["county"]))
		// 	.range([0, innerHeight])
		// 	.paddingInner(0.15);

		// let max = 0;
		// ptr_ground.forEach(p => {
		// 	max = Math.max(max,p.use)
		// })
		// xScale = d3.scaleLinear()
		// 	.domain([0, max])
		// 	.range([0, innerWidth]);
		


	}
	  $: {
    	console.log(data);
    	updateGraph();
    // create_initial();
  	}
</script>

<main>
	<h2>Ground Water Usage by County</h2>
	<!-- <svg {width} {height} id ="stacked"></svg> -->
	<svg id = "bar" {width} {height}>
		<!-- <g transform={`translate(${margin.left},${margin.top})`}>
			{#each xScale.ticks() as tickValue}
				<g transform={`translate(${xScale(tickValue)},0)`}>
					<line y2={innerHeight} stroke="black" />
					<text text-anchor="middle" dy="0.7em" y={innerHeight + 3}>
					{tickValue}
					</text>
				</g>
			{/each}
			{#each ptr_ground as data, i}
				<text
				text-anchor="middle"
				x="-60"
				dy=".3em"
				y={yScale(data.county) + yScale.bandwidth() / 2}
				>
				{data.county}
				</text>
				<rect
				x={0}
				y={yScale(data.county)}
				width={xScale(data.use)}
				height={yScale.bandwidth()}
				style="fill:skyblue"
				/>
			{/each}
	  	</g> -->
	</svg>
  </main>
  <style>
	/* rect{
		color:green;
	} */
  </style>