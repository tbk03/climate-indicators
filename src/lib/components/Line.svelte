<script>
  export let xScale;
  export let yScale;
  export let xAccessor;
  export let yAccessor;
  export let data;
  export let color = "black";

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

<path d={line_gen} stroke={color}/>

<style>
    path {
        fill: transparent;
    }
</style>
