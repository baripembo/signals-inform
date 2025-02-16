<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";

  export let data = [];

  let chartRef;
  let minDate, maxDate;

  // Define indicators with labels and colors
  const indicators = [
    { key: "inform_severity_index", label: "Inform Severity Index", color: "#5D3779" },
    { key: "impact_crisis", label: "Impact Crisis", color: "#3BBEAA" },
    { key: "people_condition", label: "People Condition", color: "#B12789" },
    { key: "complexity", label: "Complexity", color: "#C8B298" }
  ];

  onMount(() => {
    if (data.length > 0) {
      renderHeatmap();
    }
  });

  function renderHeatmap() {
    // Filter data for the selected country
    const filteredData = data
      .map(d => ({
        date: new Date(d.date),
        country: d.country,
        inform_severity_index: +d.inform_severity_index,
        impact_crisis: +d.impact_crisis,
        people_condition: +d.people_condition,
        complexity: +d.complexity
      }))
      .sort((a, b) => a.date - b.date);

    // Extract min and max date for X-axis
    [minDate, maxDate] = d3.extent(filteredData, d => d.date);

    // Define chart dimensions
    const margin = { top: 0, right: 20, bottom: 50, left: 150 };
    const width = 1200 - margin.left - margin.right;
    const height = 98 - margin.top - margin.bottom;
    const gridSize = 10;

    // Flatten data for heatmap
    const heatmapData = indicators.flatMap(({ key }) =>
      filteredData.map(d => ({
        date: d.date,
        indicator: key,
        value: d[key]
      }))
    );

    // Define scales
    // const xScale = d3.scaleTime()
    //   .domain([minDate, maxDate])
    //   .range([0, width]);

    // Define xScale based on the number of unique dates
    const uniqueDates = [...new Set(heatmapData.map(d => d.date))];
    const xScale = d3.scaleBand()
      .domain(uniqueDates)
      .range([0, width])
      .padding(0.1);

    const yScale = d3.scaleBand()
      .domain(indicators.map(d => d.key)) // Use indicator keys for positioning
      .range([0, height])
      .padding(0.1);

    // Create a unique color scale for each indicator
    const colorScales = {};
    //const minValue = d3.min(d => d.value);
    indicators.forEach(({ key, color }) => {
      colorScales[key] = d3.scaleLinear()
        .domain([3, d3.max(heatmapData.filter(d => d.indicator === key), d => d.value)])
        .range(["#BB8EC0", "#7C4192", "#5C3578", "#433557"]); // Light to assigned indicator color
    });

    // colorScales["inform_severity_index"] = ["#FFF", "#BB8EC0", "#7C4192", "#5C3578", "#433557"];
    // console.log(colorScales)

    // Append SVG element
    const svg = d3.select(chartRef)
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    // Add X-axis
    svg.append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(xScale)
        .tickValues([...new Set(heatmapData.map(d => d3.timeYear(d.date)))])
        .tickSize(0).tickPadding(8).tickFormat(d3.timeFormat("%Y")));

    // Add Y-axis with custom labels
    svg.append("g")
      .call(d3.axisLeft(yScale).tickSize(0).tickPadding(6)
        .tickFormat(d => indicators.find(ind => ind.key === d)?.label || d)
      );

    // Add heatmap cells
    svg.selectAll(".heatmap-rect")
      .data(heatmapData)
      .enter()
      .append("rect")
      .attr("class", "heatmap-rect")
      .attr("x", d => xScale(d.date))
      .attr("y", d => yScale(d.indicator))
      .attr("width", gridSize)
      .attr("height", gridSize)
      .attr("fill", (d) => {
        console.log(d.value, colorScales[d.indicator](d.value))
        return colorScales[d.indicator](d.value)
      }); // Use correct indicator color scale
  

    // // Add color legend
    // const legendWidth = 200, legendHeight = 10;
    // const legendScale = d3.scaleLinear()
    //   .domain([2, d3.max(heatmapData, d => d.value)])
    //   .range([0, legendWidth]);

    // const legendAxis = d3.axisBottom(legendScale)
    //   .tickValues(d3.range(2, Math.ceil(d3.max(heatmapData, d => d.value)) + 1))
    //   .tickFormat(d3.format(".1f"));

    // const legend = svg.append("g")
    //   .attr("transform", `translate(${width - legendWidth}, -30)`);

    // legend.selectAll("rect")
    //   .data(d3.range(legendWidth))
    //   .enter()
    //   .append("rect")
    //   .attr("x", (_, i) => i)
    //   .attr("y", 0)
    //   .attr("width", 1)
    //   .attr("height", legendHeight)
    //   .attr("fill", (_, i) => valueColorScale(legendScale.invert(i)));

    // legend.append("g")
    //   .attr("transform", `translate(0, ${legendHeight})`)
    //   .call(legendAxis);
  }
</script>


<div bind:this={chartRef} class="chart"></div>


<style>
  .chart {
    display: flex;
    justify-content: flex-start;
    align-items: flex-center;
  }
</style>
