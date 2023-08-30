<script>
  export let xScale;
  export let yScale;
  export let xAccessor;
  export let yAccessor;
  export let data;
  export let color = "black";
  export let hoveredEvent;

  // D3.js
  import { line, curveNatural } from "d3-shape";

  // -----------------------------------------------------------------------------
  // LINE GENERATOR
  // -----------------------------------------------------------------------------

  $: line_gen = line()
    .curve(curveNatural)
    // @ts-ignore
    .x((d) => xScale(xAccessor(d)))
    .y((d) => yScale(yAccessor(d)))(data);
</script>

<path
  class="line"
  d={line_gen}
  stroke={color}
  opacity={hoveredEvent ? 0.5 : 1}
/>

<style>
  path {
    fill: transparent;
    transition-property: opacity;
    transition-duration: 1500ms;
    transition-timing-function: cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }
</style>
