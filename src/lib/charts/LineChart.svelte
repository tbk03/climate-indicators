<script>
  // D3
  import { csv } from "d3-fetch";
  import { scaleLinear } from "d3-scale";
  import { extent, max } from "d3-array";

  // tidy-js
  import { tidy, mutate, filter } from "@tidyjs/tidy";
  import AxisX from "../components/AxisX.svelte";
  import AxisY from "../components/AxisY.svelte";
  import Line from "../components/Line.svelte";
  import Tooltip from "../components/Tooltip.svelte";
  import Annotation from "../components/Annotation.svelte";

  // -----------------------------------------------------------------------------
  // IMPORT AND CLEAN DATA FROM
  // -----------------------------------------------------------------------------

  let data = [];
  loadData();

  // LOAD FROM GITHUB
  async function loadData() {
    const response = await csv(
      "https://raw.githubusercontent.com/ClimateIndicator/GHG-Emissions-Assessment/main/results/ghg_emissions_co2e.csv"
    );
    data = await processData(response);
  }

  // CLEAN DATA USING tidy.js
  async function processData(data) {
    const processed = tidy(
      data,
      // data quality poor before 1959
      // @ts-ignore
      filter((d) => d.year >= 1959),
      mutate({
        // convert strings to numbers
        // @ts-ignore
        year: (d) => +d.year,
        "F-gases": (d) => +d["F-gases"],
        N2O: (d) => +d["N2O"],
        CH4: (d) => +d["CH4"],
        "CO2-LULUCF": (d) => +d["CO2-LULUCF"],
        "CO2-FFI": (d) => +d["CO2-FFI"],

        // calculate total emmission each year
        total_ghg_emissions: (d) =>
          d["F-gases"] + d["N2O"] + d["CH4"] + d["CO2-LULUCF"] + d["CO2-FFI"],
      })
    );
    return processed;
  }

  // -----------------------------------------------------------------------------
  // DIMENSIONS AND LAYOUT
  // -----------------------------------------------------------------------------
  let width = 400;
  let height = 400;

  const margin = {
    top: 20,
    right: 200,
    bottom: 40,
    left: 50,
  };

  $: innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;

  // -----------------------------------------------------------------------------
  // ACCESSORS
  // -----------------------------------------------------------------------------
  const xAccessor = (d) => d.year;
  const yAccessor = (d) => d.total_ghg_emissions;
  // -----------------------------------------------------------------------------
  // SCALES
  // -----------------------------------------------------------------------------

  $: xScale = scaleLinear()
    .domain(extent(data, (d) => xAccessor(d)))
    .range([0, innerWidth]);

  $: yScale = scaleLinear()
    .domain([0, max(data, (d) => yAccessor(d))])
    .range([innerHeight, 0]);

  // -----------------------------------------------------------------------------
  // TOOLTIP
  // -----------------------------------------------------------------------------
  let hoveredEvent = null;
  // $: console.log(hoveredEvent);

  let mousePosition = {};
  let positionOnChart = {};
  $: {
    // calculate mouse position from event
    if (hoveredEvent) {
      mousePosition.x = hoveredEvent.layerX - margin.left;
      mousePosition.y = hoveredEvent.layerY - margin.top;

      let nearestYear = Math.round(xScale.invert(mousePosition.x));
      positionOnChart = {
        year: nearestYear,
        total_ghg_emissions: yAccessor(data.find(d=> xAccessor(d) === nearestYear)),
      };

      // console.log(mousePosition);
    }
  }

</script>

<!-- while waiting for data to load hold the space -->
{#if data.length === 0}
  <svg {width} {height} />
  <!-- then create the chart -->
{:else}
  <div class="chart-container" bind:clientWidth={width}>
    <svg {width} {height}>
      <!-- apply top and left margins -->
      <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
        <AxisY {yScale} {xScale} width={innerWidth} {hoveredEvent} />

        {#if !hoveredEvent}
          <AxisX {xScale} height={innerHeight} width={innerWidth} />
        {/if}

        <!-- {#each data as d}
          <circle
            cx={xScale(xAccessor(d))}
            cy={yScale(yAccessor(d))}
            r={1}
            fill="purple"
            stroke="black"
            stroke-width={1}
          />
        {/each} -->

        <Line {xScale} {yScale} {xAccessor} {yAccessor} {data} {hoveredEvent} />

        {#if hoveredEvent}
          <Tooltip
            {positionOnChart}
            {xScale}
            {yScale}
            {data}
            {xAccessor}
            {yAccessor}
          />
        {/if}

        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <!-- Event listener layer -->
        <rect
          x={0}
          y={0}
          width={innerWidth}
          height={innerHeight}
          fill="transparent"
          on:mouseover={(e) => (hoveredEvent = e)}
          on:mousemove={(e) => (hoveredEvent = e)}
          on:mouseleave={() => (hoveredEvent = null)}
          on:mouseout={() => {
            hoveredEvent = null;
          }}
        />
      </g>
    </svg>
    <Annotation
       {data} {xScale} {yScale} {xAccessor} {yAccessor} {margin} {positionOnChart}
    />
  </div>
{/if}

<p>Line Chart</p>

<style>
  :global(.tick text, .axis-title) {
    font-weight: 400; /* How thick our text is */
    font-size: 12px; /* How big our text is */
    fill: hsla(212, 10%, 53%, 1); /* The color of our text */
  }

  .chart-container {
    max-width: 1000px;
    position: relative;
  }
</style>
