<script>
  export let year;

  // ----------------------------------------------------------------------
  // CHART DIMENSIONS
  // ----------------------------------------------------------------------

  let chartWidth, innerWidth;
  let margin = { top: 30, right: 10, bottom: 100, left: 10 };

  $: {
    innerWidth = chartWidth - margin.right - margin.left;
  }

  // ----------------------------------------------------------------------
  // SCALES
  // ----------------------------------------------------------------------
  let xYear1, xYear2;

  $:{
    xYear1 = 0.2 * innerWidth;
    xYear2 = 0.7 * innerWidth;
  }
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
    <svg height="600" width={chartWidth}>
      <g transform="translate({margin.left}, {margin.top})">
        <text x={xYear1} y="0" dy="-10">{comparisonYear}</text>
        <text x={xYear2} y="0" dy="-10">{year}</text>
        <line
          x1={xYear1}
          x2={xYear1}
          y1="0"
          y2="600"
          stroke="black"
        />
        <line
          x1={xYear2}
          x2={xYear2}
          y1="0"
          y2="600"
          stroke="black"
        />
      </g>
    </svg>
  </div>
</div>
