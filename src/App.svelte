<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import LineChart from "./lib/LineChart.svelte";
  import Heatmap from "./lib/Heatmap.svelte";
  import SmallMultiples from "./lib/SmallMultiples.svelte";

  let data = [];

  onMount(async () => {
    const csvData = await d3.csv('inform_severity_full_dataset.csv');
    console.log('app data', csvData);
    data = csvData;
  });
</script>

{#if data.length > 0}
  <LineChart {data} />
  <br><br>
  <SmallMultiples {data} />
{/if}
