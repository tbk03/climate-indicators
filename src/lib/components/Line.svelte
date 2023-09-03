<script>
  export let xScale;
  export let yScale;
  export let xAccessor;
  export let yAccessor;
  export let data;
  export let color = "#C94A54";
  export let hoveredEvent;

  // D3.js
  import { line, area, curveNatural } from "d3-shape";

  // -----------------------------------------------------------------------------
  // LINE GENERATOR
  // -----------------------------------------------------------------------------

  $: lineGen = line()
    .curve(curveNatural)
    // @ts-ignore
    .x((d) => xScale(xAccessor(d)))
    .y((d) => yScale(yAccessor(d)))(data);

  $: areaGen = area()
    .curve(curveNatural)
    // @ts-ignore
    .x((d) => xScale(xAccessor(d)))
    .y0(yScale(0))
    .y1((d) => yScale(yAccessor(d)))(data);
</script>

<path
  d={areaGen}
  fill="url(#area-gradient)"
  stroke="none"
  opacity={hoveredEvent ? 0.15 : 0.3}
/>
<path
  class="line"
  d={lineGen}
  stroke={color}
  stroke-width={3}
  opacity={hoveredEvent ? 0.35 : 1}
/>
<defs>
  <linearGradient id="area-gradient" x1="0" x2="0" y1="0" y2="1">
    <stop offset="0%" stop-color="#D19499" />
    <stop offset="100%" stop-color="#F8F1F4" />
  </linearGradient>
</defs>

<style>
  .line {
    fill: transparent;
    transition-property: opacity;
    transition-duration: 1500ms;
    transition-timing-function: cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }
</style>
