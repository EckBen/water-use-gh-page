<script lang="ts">
	import {onMount} from 'svelte';
	import * as d3 from "d3";
    import { easeCircle } from 'd3';
    // import App from 'src/App.svelte';
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
		let counties = [];
		for (let idx = 1; idx < data.length; idx++) {
			let ground_this = {};
			let county = data[idx][2].split(' ');
			ground_this['county'] = county[0];
			counties.push(county[0]);

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

		// convert to percentages
		for (let idx = 0; idx < ptr_ground.length; idx++){
			let curr_county = ptr_ground[idx];
			// console.log(curr_county);
			for (let i = 0; i < cols_names.length; i++){
				// console.log(curr_county[cols_names[i]]);
				curr_county[cols_names[i]] = parseFloat((curr_county[cols_names[i]] / curr_county.use * 100).toFixed(2));
				
			}
			curr_county.use = 100.00;
		}

		console.log(ptr_ground);

		let abbrev_to_use = new Map([
			['AQ-WGWTo' , 'Public Supply Ground Water Withdrawal'],
			['DO-WGWFr' , 'Domestic Self-supplied Ground Water Withdrawal'],
			['IN-WGWTo' , 'Industrial Self-supplied Ground Water Withdrawal'],
			['IR-WGWFr' , 'Irrigation Ground Water Withdrawal'],
			['LI-WGWFr' , 'Livestock Ground Water Withdrawal'],
			['MI-WGWTo' , 'Aquaculture Ground Water Withdrawal'],
			['PS-GWPop' , 'Mining Ground Water Withdrawal']
		])

		// const ptr_ground= [
		// 	{county: 'Jenny', A:40, B:10, C:25, D:25,total:100},
		// 	{county: 'Maggie', A:28, B:42, C:5, D:25,total: 100},
		// 	{county: 'Stephanie', A:10, B:13, C:37, D:40,total: 100}
		// ]

		let county_names = ptr_ground.map(c => c.county);

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
                            .domain(county_names)
                            .range([0, chartWidth_bar])
                            .padding(0.05);

							// const yExtent = d3.extent()
                        const yScale = d3.scaleLinear()
                            .domain([0,100])
                            .range([chartHeight_bar, 0]);

                        console.log(cols_names);

						// const colorScale = d3.scaleSequential()
						// 						.domain(counties)
						// 						.range(d3.schemeCategory10);


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

							let mouseOver = function(d) {
								console.log(d);
								let this_county_id = d.target.id;
								let this_county = d.target.__data__.county;
								console.log(this_county)
								// data = data.filter(d => d.year === parseInt(sl_year));
								let this_county_data = ptr_ground.filter(d => d.county === this_county);
								let perc = this_county_data[0][this_county_id];
								console.log(perc)
								
								let this_source = abbrev_to_use.get(d.target.id);
								console.log(this_source);
								let text = this_source + '  :  ' + perc + ' % '
								// console.log();
								// console.log(d.toElement.__data__.county);
								let text_source = d3.select("#water_source");
                				text_source.select("#this_source").text(text);
							}

							let mouseLeave = function(d) {
								console.log(d);
								let text_source = d3.select("#water_source");
                				text_source.select("#this_source").text("");
								// d3.selectAll(".state")
								// 	.transition()
								// 	.duration(200)
								// 	.style("opacity", 1)
								// d3.select(this)
								// 	.transition()
								// 	.duration(200)
								// 	.style("stroke", "transparent")
							}


                        chartArea_bar.selectAll('rect.bar').data( ptr_ground)
                                .join('rect').attr('class','bars')
								.attr('id','AQ-WGWTo')
                                .attr("fill", 'steelblue')
                                .attr("x", d => xScale(d['county']))
                                .attr("y", d => yScale(d['AQ-WGWTo']))
                                .attr("height", d => yScale(0) - yScale(d['AQ-WGWTo']))
                                .attr("width", xScale.bandwidth())
								.on('mouseover', mouseOver)
            					.on('mouseout', mouseLeave);

                        chartArea_bar.selectAll('rect.bar').data( ptr_ground)
                                .join('rect').attr('class','bars')
								.attr('id','DO-WGWFr')
                                .attr("fill", 'aqua')
                                .attr("x", d => xScale(d['county']))
                                .attr("y", d => yScale(d['AQ-WGWTo'] + d['DO-WGWFr']))
                                .attr("height", d => yScale(d['AQ-WGWTo']) - yScale(d['AQ-WGWTo'] + d['DO-WGWFr']))
                                .attr("width", xScale.bandwidth())
								.on('mouseover', mouseOver)
            					.on('mouseout', mouseLeave);

						chartArea_bar.selectAll('rect.bar').data( ptr_ground)
                                .join('rect').attr('class','bars')
								.attr('id','IN-WGWTo')
                                .attr("fill", 'dodgerblue')
                                .attr("x", d => xScale(d['county']))
                                .attr("y", d => yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo']))
                                .attr("height", d => yScale(d['AQ-WGWTo'] + d['DO-WGWFr']) - yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo']))
                                .attr("width", xScale.bandwidth())
								.on('mouseover', mouseOver)
            					.on('mouseout', mouseLeave);

						chartArea_bar.selectAll('rect.bar').data( ptr_ground)
                                .join('rect').attr('class','bars')
								.attr('id','IR-WGWFr')
                                .attr("fill", 'cadetblue')
                                .attr("x", d => xScale(d['county']))
                                .attr("y", d => yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo'] + d['IR-WGWFr']))
                                .attr("height", d => yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo']) - yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo'] + d['IR-WGWFr']))
                                .attr("width", xScale.bandwidth())
								.on('mouseover', mouseOver)
            					.on('mouseout', mouseLeave);
						
						chartArea_bar.selectAll('rect.bar').data( ptr_ground)
                                .join('rect').attr('class','bars')
								.attr('id','LI-WGWFr')
                                .attr("fill", 'cornflowerblue')
                                .attr("x", d => xScale(d['county']))
                                .attr("y", d => yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo'] + d['IR-WGWFr'] + d['LI-WGWFr']))
                                .attr("height", d => yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo'] + d['IR-WGWFr']) - yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo'] + d['IR-WGWFr'] + d['LI-WGWFr']))
                                .attr("width", xScale.bandwidth())
								.on('mouseover', mouseOver)
            					.on('mouseout', mouseLeave);

						chartArea_bar.selectAll('rect.bar').data( ptr_ground)
                                .join('rect').attr('class','bars')
								.attr('id','MI-WGWTo')
                                .attr("fill", 'cyan')
                                .attr("x", d => xScale(d['county']))
                                .attr("y", d => yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo'] + d['IR-WGWFr'] + d['LI-WGWFr'] + d['MI-WGWTo']))
                                .attr("height", d => yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo'] + d['IR-WGWFr'] + d['LI-WGWFr']) - yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo'] + d['IR-WGWFr'] + d['LI-WGWFr'] + d['MI-WGWTo']))
                                .attr("width", xScale.bandwidth())
								.on('mouseover', mouseOver)
            					.on('mouseout', mouseLeave);
						
						chartArea_bar.selectAll('rect.bar').data(ptr_ground)
                                .join('rect')
								.attr('class','bars')
								.attr('id', 'PS-GWPop')
                                .attr("fill", 'lavender')
                                .attr("x", d => xScale(d['county']))
                                .attr("y", d => yScale(d['use']))
                                .attr("height", d => yScale(d['AQ-WGWTo'] + d['DO-WGWFr'] + d['IN-WGWTo'] + d['IR-WGWFr'] + d['LI-WGWFr'] + d['MI-WGWTo']) - yScale(d['use']))
                                .attr("width", xScale.bandwidth())
								.on('mouseover', mouseOver)
            					.on('mouseout', mouseLeave);
                        
                        chartArea_bar.selectAll('text').data( ptr_ground )
                                .join('text')
                                .attr('class', 'labels')
                                .text(d => d['county'])
                                .attr('x', 0)
                                .attr("y", 0)
                                .attr('transform', d => `translate(${xScale(d['county']) + xScale.bandwidth()/2},${yScale(d['use']) - 10}) rotate(-60)`)
								// .on('mouseover', mouseOver)
            					// .on('mouseout', mouseLeave);

						

						// chartArea_bar.selectAll('rect.bar')
						// 				.on('mouseover', mouseOver)
            			// 				.on('mouseout', mouseLeave);
                    
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
	<div class = "bar_plot">
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
		<div >
			<table id="table">
				<th id="thead"></th>
				<tr>
					<td id="water_source">
						<p id="this_source"></p>
					</td>
				</tr>
			</table>
		</div>
	</div>
  </main>
  <style>
	/* rect{
		color:green;
	} */
	.bar_plot{
		display: flex;
		flex-direction: column;
		align-items: center;
	}
  </style>