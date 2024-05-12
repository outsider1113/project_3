<script lang="ts">
    import { extent, rollup } from "d3-array";
    import { csv, json } from "d3-fetch";
    import { geoPath } from "d3-geo";
    import { scaleQuantile, scaleLinear } from "d3-scale";
    import { type UsAtlas, mesh, feature } from "topojson";
    
    export let datasets = [];
    export let colors = [];

    import { onMount } from "svelte";
    const path = geoPath(); 


    let stateMesh = null;
let statesFeatures = [];
let dimensions = {
  width: 975,
  height: 720,
  margin: {
    top: 24,
    right: 0,
    left: 0,
    bottom: 6
  }
};
onMount(async () => {
  const counts = await Promise.all(
    datasets.map(
      async ({ url }) =>
        await csv(url, ({ state, outages_count }) => ({
          state,
          outages_count: +outages_count,
        }))
    )
  );

  const data = [...counts[0], ...counts[1]];

  const us = await json("/data/us_topojson.json");

  stateMesh = topojson.mesh(us, us.objects.states, (a, b) => a !== b);

  statesFeatures = topojson.feature(us, us.objects.states).features;
});


  </script>

  <main>
    <svg width={dimensions.width} height={dimensions.height} viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}>
    </svg>

    <g>
      <path fill="none" stroke="white" stroke-linejoin="round" d={path(stateMesh)} />
      {#each statesFeatures as feature}
        <path fill="green" d={path(feature)}/>
      {/each}
    </g>
  </main>