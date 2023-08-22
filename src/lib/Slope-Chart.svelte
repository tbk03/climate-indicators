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
  let xYear2Prop = 0.5;

  $: xYear1 = xYear1Prop * innerWidth;
  $: xYear2 = xYear2Prop * innerWidth;

  // Y SCALE
  let dummyDataY1 = { emissons: 0 };
  let dummyDataY2 = { emissons: 9 };
  let yScale = scaleLinear().domain([10, 0]).range([0, innerHeight]);

  // ----------------------------------------------------------------------
  // COMPARE THE TWO YEARS
  // ----------------------------------------------------------------------
  let comparisonOffset = 10;
  $: comparisonYear = year - comparisonOffset;
</script>

<div class="waffle-title chart-title">
  <h1>Between {comparisonYear} and {year} GHG emissions rose by ...%</h1>
</div>
<div class="slope-container chart-container">
  <div class="slope chart" bind:clientWidth={chartWidth}>
    <svg height={chartHeight} width={chartWidth}>
      <g transform="translate({margin.left}, {margin.top})">
        <!-- vertical reference line labels -->
        <text x={xYear1} y="0" dy="-10" class="year-label"
          >{comparisonYear}
        </text>
        <text x={xYear2} y="0" dy="-10" class="year-label">{year}</text>

        <!-- vertical reference lines -->
        <line
          class="vert-ref-line"
          x1={xYear1}
          x2={xYear1}
          y1="0"
          y2={innerHeight}
          stroke="black"
        />
        <line
          class="vert-ref-line"
          x1={xYear2}
          x2={xYear2}
          y1="0"
          y2={innerHeight}
          stroke="black"
        />

        <!-- Slope -->
        <circle cx={xYear1} cy={yScale(dummyDataY1.emissons)} r="10" />
        <circle cx={xYear2} cy={yScale(dummyDataY2.emissons)} r="10" />
        <line
          class="slope-line"
          x1={xYear1}
          y1={yScale(dummyDataY1.emissons)}
          x2={xYear2}
          y2={yScale(dummyDataY2.emissons)}
          stroke="black"
        />
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
