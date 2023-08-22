<script>
  // ----------------------------------------------------------------------
  // IMPORTS
  // ----------------------------------------------------------------------
  import { scaleLinear } from "d3-scale";

  // ----------------------------------------------------------------------
  // PROPERTIES - PASS INTO THE COMPONENT
  // ----------------------------------------------------------------------
  export let year;

  // ----------------------------------------------------------------------
  // CHART DIMENSIONS
  // ----------------------------------------------------------------------

  let chartWidth = 600;
  let chartHeight = 400;
  let margin = { top: 30, right: 10, bottom: 100, left: 10 };
  $: innerWidth = chartWidth - margin.right - margin.left;
  innerHeight = chartHeight - margin.top - margin.bottom;

  // ----------------------------------------------------------------------
  // SCALES
  // ----------------------------------------------------------------------

  // X SCALE

  // how far across the svg should the vertical reference lines be
  // a proportion of svg width
  let xYear1Prop = 0.2;
  let xYear2Prop = 0.6;

  $: xYear1 = xYear1Prop * innerWidth;
  $: xYear2 = xYear2Prop * innerWidth;

  // Y SCALE
  let yScale = scaleLinear().domain([10, 0]).range([0, innerHeight]);

  // ----------------------------------------------------------------------
  // COMPARE THE TWO YEARS
  // ----------------------------------------------------------------------
  let comparisonOffset = 10;
  $: comparisonYear = year - comparisonOffset;

  // ----------------------------------------------------------------------
  // DUMMY DATA
  // ----------------------------------------------------------------------
  $: dummyData = [
    { x: xYear1, y: 2, year: year },
    { x: xYear2, y: 7, year: comparisonYear },
  ];
</script>

<div class="waffle-title chart-title">
  <h1>Between {comparisonYear} and {year} GHG emissions rose by ...%</h1>
</div>
<div class="slope-container chart-container">
  <div class="slope chart" bind:clientWidth={chartWidth}>
    <svg height={chartHeight} width={chartWidth}>
      <g transform="translate({margin.left}, {margin.top})">
        <!-- CORE CHART -->
        {#each dummyData as d}
          <!-- vertical reference lines -->
          <text x={d.x} y="0" dy="-10" class="year-label">{d.year} </text>
          <line
            class="vert-ref-line"
            x1={d.x}
            x2={d.x}
            y1="0"
            y2={innerHeight}
            stroke="black"
          />
          <!-- points -->
          <circle cx={d.x} cy={yScale(d.y)} r="10" />
        {/each}

        <!-- Slope -->
        <line
          class="slope-line"
          x1={dummyData[0].x}
          y1={yScale(dummyData[0].y)}
          x2={dummyData[1].x}
          y2={yScale(dummyData[1].y)}
          stroke="black"
        />

        <!-- ANNOTATIONS -->
        <!-- horizontal reference lines -->
        {#each dummyData as d}
          <line
            class="horz-ref-line"
            x1={d.x}
            y1={yScale(d.y)}
            x2={(xYear2Prop + 0.05) * innerWidth}
            y2={yScale(d.y)}
            stroke="black"
          />
        {/each}
        <!-- arrow -->

      </g>
    </svg>
  </div>
</div>

<style>
  /* REFERENCE LINES - YEARS */
  .year-label {
    text-anchor: middle;
  }
</style>
