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


<path d={areaGen} fill="#EAC8CA" stroke="none" opacity="0.3"/>
<path
  class="line"
  d={lineGen}
  stroke={color}
  stroke-width={3}
  opacity={hoveredEvent ? 0.2 : 1}
/>

<style>
  .line {
    fill: transparent;
    transition-property: opacity;
    transition-duration: 1500ms;
    transition-timing-function: cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }
</style>
