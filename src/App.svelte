<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import LineChart from "./lib/LineChart.svelte";
  import SmallMultiples from "./lib/SmallMultiples.svelte";

  let data = [];

  // get location from url
  let url = window.location.href;
  let params = new URLSearchParams(window.location.search);
  let location = params.get('location');
  let iso3 = location && location.trim() !== '' ? location.toUpperCase() : 'COL';

  onMount(async () => {
    const csvData = await d3.csv('inform_severity_full_dataset.csv');
    data = csvData.filter(row => row.iso3 === iso3);
    console.log(iso3, data);
  });
</script>

{#if data.length > 0}
  <h2><strong>{data[0].country}</strong></h2>
  <LineChart {data} />
  <br><br>

  <h2><strong>{data[0].country}</strong></h2>
  <SmallMultiples {data} />
{/if}
