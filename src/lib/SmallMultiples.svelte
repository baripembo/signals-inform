<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  export let data = [];

  let chartRef;
  let minDate, maxDate;

  onMount(() => {
    if (data.length > 0) {
      renderCharts();
    }
  });

  function renderCharts() {
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

    // Extract min and max date for the X-axis
    [minDate, maxDate] = d3.extent(filteredData, d => d.date);

    // Define chart dimensions
    const margin = { top: 30, right: 25, bottom: 40, left: 40 };
    const width = 300 - margin.left - margin.right;
    const height = 160 - margin.top - margin.bottom;

    // Define indicators
    const indicators = [
      { key: "inform_severity_index", label: "Inform Severity Index", color: "#5D3779" },
      { key: "impact_crisis", label: "Impact Crisis", color: "#3BBEAA" },
      { key: "people_condition", label: "People Condition", color: "#B12789" },
      { key: "complexity", label: "Complexity", color: "#C8B298" }
    ];

    // Define scales
    const xScale = d3.scaleTime().domain([minDate, maxDate]).range([0, width]);

    const yScale = d3.scaleLinear()
      .domain([2, 5]) //d3.max(filteredData, d => Math.max(d.inform_severity_index, d.impact_crisis, d.people_condition, d.complexity))]
      .nice()
      .range([height, 0]);

    // Clear previous charts
    d3.select(chartRef).selectAll('*').remove();

    // Append an SVG container for each small multiple
    const container = d3.select(chartRef)
      .append("div")
      .attr("class", "small-multiples")
      .style("display", "grid")
      .style("grid-template-columns", "repeat(2, 1fr)") // 2 columns layout
      .style("gap", "20px");

    indicators.forEach(({ key, label, color }) => {
      const svg = container.append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

      // Background bands
      const bands = [
        { min: 2, max: 3, color: "#fff" },
        { min: 3, max: 4, color: "#fff" },
        { min: 4, max: 5, color: "#FCE0DE" }
      ];

      svg.selectAll(".background-band")
        .data(bands)
        .enter()
        .append("rect")
        .attr("x", 0)
        .attr("y", d => yScale(d.max))
        .attr("width", width)
        .attr("height", d => yScale(d.min) - yScale(d.max))
        .attr("fill", d => d.color)
        .attr("opacity", 0.5);

      // X-axis
      svg.append("g")
        .attr("transform", `translate(0,${height})`)
        .call(d3.axisBottom(xScale).ticks(4).tickFormat(d3.timeFormat("%Y")).tickSize(0));

      // Y-axis
      svg.append("g")
        .call(d3.axisLeft(yScale).ticks(4).tickPadding(8).tickSize(-width).tickFormat(d3.format(".0f")));

      // Line generator
      const line = d3.line()
        .x(d => xScale(d.date))
        .y(d => yScale(d[key]))
        .curve(d3.curveLinear);

      // Line path
      svg.append("path")
        .datum(filteredData)
        .attr("fill", "none")
        .attr("stroke", color)
        .attr("stroke-width", 2)
        .attr("d", line);


      // Find last data point
      const lastDataPoint = filteredData[filteredData.length - 1];

      // Add label for last value
      svg.append("text")
        .attr("x", xScale(lastDataPoint.date) + 5) // Slightly to the right
        .attr("y", yScale(lastDataPoint[key]) + 3) // Adjust positioning
        .style("fill", color)
        .style("font-size", "14px")
        .style("font-weight", "bold")
        .text(`${lastDataPoint[key].toFixed(1)}`);

      // Add highest and lowest points
      const maxDataPoint = d3.max(filteredData, d => d[key]);
      const minDataPoint = d3.min(filteredData, d => d[key]);

      const maxPoint = filteredData.find(d => d[key] === maxDataPoint);
      const minPoint = filteredData.find(d => d[key] === minDataPoint);

      svg.selectAll(".dot")
        .data([maxPoint, minPoint, lastDataPoint])
        .enter()
        .append("circle")
        .attr("cx", d => xScale(d.date))
        .attr("cy", d => yScale(d[key]))
        .attr("r", 4)
        .attr("fill", color)
        .attr("stroke", "white")
        .attr("stroke-width", 1);

      svg.selectAll(".dot-label")
        .data([maxPoint, minPoint])
        .enter()
        .append("text")
        .attr("x", d => xScale(d.date) - 7)
        .attr("y", d => d === maxPoint ? yScale(d[key]) - 7 : yScale(d[key]) + 15)
        .style("fill", color)
        .style("font-size", "12px")
        .text(d => d[key].toFixed(1));

      // Add title for each chart
      svg.append("text")
        .attr("x", width / 2)
        .attr("y", -15)
        .attr("text-anchor", "middle")
        .style("font-size", "14px")
        .text(label);
    });
  }
</script>


<div bind:this={chartRef} class="chart"></div>


<style>
  .chart {
    display: flex;
    justify-content: flex-start;
    align-items: center;
  }
  h2 {
    margin-bottom: 20px;
  }
</style>
