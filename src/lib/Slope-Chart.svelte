<script>
  export let year;

  let chartWidth, innerWidth;
  let margin = { top: 30, right: 10, bottom: 100, left: 10 };
  let comparisonOffset = 10;
  
  $: comparisonYear = year - comparisonOffset;

  $: {
    innerWidth = chartWidth - margin.right - margin.left;
    console.log(innerWidth);
    }
</script>

<div class="waffle-title chart-title">
  <h1>Between {comparisonYear} and {year} GHG emissions rose by ...%</h1>
</div>
<div class="slope-container chart-container">
  <div class="slope chart" bind:clientWidth={chartWidth}>
    <svg height="600" width={chartWidth}>
        <g transform="translate({margin.left}, {margin.top})">
            <text x={0.2 * innerWidth} y="0" dy="-10">{comparisonYear}</text>
            <line x1={0.2 * innerWidth} x2={0.2*innerWidth} y1="0" y2="600" stroke="black"/>
            <line x1={0.7 * innerWidth} x2={0.7*innerWidth} y1="0" y2="600" stroke="black"/>
        </g>
        
    </svg>
  </div>
</div>
