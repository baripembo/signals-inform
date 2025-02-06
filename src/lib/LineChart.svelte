<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  export let data = [];
  export let iso3 = "COL";

  let chartRef;
  let minDate, maxDate;
  let title;
  let filteredData;

  onMount(() => {
    if (data.length > 0) {
      renderChart();
    }
  });

  function renderChart() {
    // Filter data for the selected country
    filteredData = data
      .filter(d => d.iso3 === iso3)
      .map(d => ({
        date: new Date(d.date),
        country: d.country,
        inform_severity_index: +d.inform_severity_index,
        impact_crisis: +d.impact_crisis,
        people_condition: +d.people_condition,
        complexity: +d.complexity
      }))
      .sort((a, b) => a.date - b.date);

    title = filteredData[0].country;

    // Extract min and max date for the X-axis
    [minDate, maxDate] = d3.extent(filteredData, d => d.date);

    // Define chart dimensions
    const margin = { top: 20, right: 150, bottom: 50, left: 50 };
    const width = 650 - margin.left - margin.right;
    const height = 300 - margin.top - margin.bottom;

    // Define indicators and create line paths
    const indicators = ["inform_severity_index", "impact_crisis", "people_condition", "complexity"];

    // Append SVG element
    const svg = d3.select(chartRef)
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    // Create scales
    const xScale = d3.scaleTime()
      .domain([minDate, maxDate])
      .range([0, width]);

    const yScale = d3.scaleLinear()
      .domain([2, d3.max(filteredData, d => Math.max(d.inform_severity_index, d.impact_crisis, d.people_condition, d.complexity))])
      .nice()
      .range([height, 0]);

    const colorScale = d3.scaleOrdinal()
      .domain(indicators)
      .range(['#5D3779', '#3BBEAA', '#B12789', '#C8B298']); 

    // ** ADD BACKGROUND BANDS **
    const bands = [
      { min: 2, max: 3, color: "#FFF" }, //F4F4F4
      { min: 3, max: 4, color: "#FFF" }, //EBEBEB
      { min: 4, max: 5, color: "#FCE0DE" }
    ];

    svg.selectAll(".background-band")
      .data(bands)
      .enter()
      .append("rect")
      .attr("class", "background-band")
      .attr("x", 0)
      .attr("y", d => yScale(d.max)) 
      .attr("width", width)
      .attr("height", d => yScale(d.min) - yScale(d.max)) // Height spans value range
      .attr("fill", d => d.color)
      .attr("opacity", 0.5); 

    // Add X-axis
    svg.append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(xScale).tickSize(0).tickPadding(8).ticks(6).tickFormat(d3.timeFormat("%Y")));

    // Add Y-axis
    svg.append("g")
      .call(d3.axisLeft(yScale).tickPadding(8).ticks(5).tickSize(0));

    // Y-axis grid lines
    const yGrid = d3.axisLeft(yScale)
      .tickSize(-width) 
      .ticks(3)
      .tickFormat('');

    // Append the grid lines to the SVG
    svg.append('g')
      .attr('class', 'grid')
      .call(yGrid);

    // Define line generator
    const line = d3.line()
      .curve(d3.curveLinear)
      .x(d => xScale(d.date));


    indicators.forEach(indicator => {
      svg.append("path")
        .datum(filteredData)
        .attr("fill", "none")
        .attr("stroke", colorScale(indicator))
        .attr("stroke-width", d => indicator=='inform_severity_index' ? 3 : 1)
        .attr("d", line.y(d => yScale(d[indicator])));
    });


    // Function to format column names
    function formatLabel(text) {
      return text
        .replace(/_/g, " ") // Replace underscores with spaces
        .replace(/\b\w/g, c => c.toUpperCase()); // Capitalize each word
    }

    // Add labels at the end of each line
    indicators.forEach(indicator => {
      const lastDataPoint = filteredData[filteredData.length - 1];

      svg.append("text")
        .attr("x", xScale(lastDataPoint.date) + 5) // Slightly offset to the right
        .attr("y", yScale(lastDataPoint[indicator]))
        .style("fill", colorScale(indicator))
        .style("font-size", "14px")
        .style("font-weight", d => indicator=='inform_severity_index' ? 700 : 400)
        .text(`${formatLabel(indicator)} ${lastDataPoint[indicator]}`);
    });
  }
</script>

<h2><strong>{title}</strong></h2>
<div bind:this={chartRef} class="chart"></div>


<style>
  .chart {
    display: flex;
    justify-content: flex-start;
    align-items: flex-center;
  }
</style>
