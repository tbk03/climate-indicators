<script>
  export let data;
  // D3
  import { scaleLinear } from "d3-scale";
  import { max, min } from "d3-array";

  // tidy-js
  import AxisX from "../components/AxisX.svelte";
  import AxisY from "../components/AxisY.svelte";
  import Line from "../components/Line.svelte";
  import Tooltip from "../components/Tooltip.svelte";
  import Annotation from "../components/Annotation.svelte";

  // fonts
  import "@fontsource/lato/400.css";
  import "@fontsource/lato/700.css";

  

  // -----------------------------------------------------------------------------
  // DIMENSIONS AND LAYOUT
  // -----------------------------------------------------------------------------
  let width = 400;
  let height = 450;

  const widthThreshold = 600;
  let marginRightWideScreen = 160;
  let marginRightNarrowScreen = 40;

  let margin = {
    top: 60,
    right: marginRightWideScreen,
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
    .domain([1958, max(data, (d) => xAccessor(d))])
    .range([0, innerWidth]);

  $: yScale = scaleLinear()
    .domain([0, max(data, (d) => yAccessor(d)) + 2])
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

      // is mouse within chart but outside the coverage of the data
      let nearestYear = Math.round(xScale.invert(mousePosition.x));
      let minYear = min(data, (d) => xAccessor(d));
      nearestYear = nearestYear >= minYear ? nearestYear : minYear;
      positionOnChart = {
        year: nearestYear,
        total_ghg_emissions: yAccessor(
          data.find((d) => xAccessor(d) === nearestYear)
        ),
      };

      // console.log(mousePosition);
    }
  }

  // -----------------------------------------------------------------------------
  // SWITCH LAYOUT BASED ON WIDTH
  // -----------------------------------------------------------------------------
  $: {
    if (width < widthThreshold) {
      margin.right = marginRightNarrowScreen;
    } else if (width >= widthThreshold) {
      margin.right = marginRightWideScreen;
    }
  }

  $: console.log({ width }, margin.right);
</script>

<!-- while waiting for data to load hold the space -->
{#if data.length === 0}
  <div class="svg-container">
    <svg {width} {height} />
  </div>
  <!-- then create the chart -->
{:else}
  <div class="chart-container" bind:clientWidth={width}>
    <div class="svg-container">
      <svg {width} {height}>
        <!-- apply top and left margins -->
        <g
          class="inner-chart"
          transform="translate({margin.left}, {margin.top})"
        >
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

          <Line
            {xScale}
            {yScale}
            {xAccessor}
            {yAccessor}
            {data}
            {hoveredEvent}
          />

          {#if hoveredEvent}
            <Tooltip
              {positionOnChart}
              {xScale}
              {yScale}
              {data}
              {xAccessor}
              {yAccessor}
              width={innerWidth}
              height={innerHeight}
            />
          {/if}

          <!-- svelte-ignore a11y-no-static-element-interactions -->
          <!-- Event listener layer -->
          <rect
            id="listener-layer"
            x={0}
            y={0}
            width={innerWidth}
            height={innerHeight}
            fill="transparent"
            on:mouseover={(e) => (hoveredEvent = e)}
            on:focus={(e) => (hoveredEvent = e)}
            on:mousemove={(e) => (hoveredEvent = e)}
            on:mouseleave={() => (hoveredEvent = null)}
            on:mouseout={() => {
              hoveredEvent = null;
            }}
            on:blur={() => {
              hoveredEvent = null;
            }}
            on:touchend={() => {
              hoveredEvent = null;
            }}
          />
        </g>
      </svg>
    </div>
    <div class="annotation-container">
      {#if hoveredEvent}
        <!-- {#if true} -->
        <Annotation
          {data}
          {xScale}
          {yScale}
          {xAccessor}
          {yAccessor}
          {margin}
          {positionOnChart}
          {width}
          {widthThreshold}
        />
      {/if}
    </div>
  </div>
{/if}

<style>
  /* SAME FONT AND COLOR - axis labels, tooltip axis references, axis tick labels */
  :global(.axis-label, .tick text, .tt-reference-text, .annotation, .units) {
    font-family: "Lato", sans-serif;
    fill: hsla(178, 16%, 13%, 1);
  }

  /* SIZES AND WEIGHTS ARE DIFFERENT */
  :global(.tick text) {
    font-weight: 400; /* How thick our text is */
    font-size: 14px; /* How big our text is */
  }

  :global(.tt-reference-text) {
    font-size: 16px;
    font-weight: 700;
  }

  .chart-container {
    max-width: 1000px;
    position: relative;
  }

  #listener-layer:focus {
    outline: none;
  }

  .svg-container {
    z-index: -1;
  }

  .annotation-container {
    z-index: -1;
  }
</style>
