<script>
  // SVELTE
  import { tweened } from "svelte/motion";
  import { cubicOut, cubicIn } from "svelte/easing";
  import { fade } from "svelte/transition";

  // D3.js
  import { max } from "d3-array";

  // LOCAL COMPONENTS
  import Arrow from "./Arrow.svelte";

  export let positionOnChart;
  export let xScale;
  export let yScale;
  export let xAccessor;
  export let yAccessor;
  export let data;

  // -----------------------------------------------------------------------------
  // SELECTING A DATAPOINT
  // -----------------------------------------------------------------------------
  let dataPoint = data.find((d) => xAccessor(d) === xAccessor(positionOnChart));

  const transitionDuration = 200;
  const tweenedTransition = {
    duration: transitionDuration,
    easing: cubicOut,
  };

  let nearestDataX = tweened(xAccessor(dataPoint), tweenedTransition);
  let nearestDataY = tweened(yAccessor(dataPoint), tweenedTransition);

  $: {
    dataPoint = data.find((d) => xAccessor(d) === xAccessor(positionOnChart));
    nearestDataX.set(xScale(xAccessor(dataPoint)));
    nearestDataY.set(yScale(yAccessor(dataPoint)));
  }

  // -----------------------------------------------------------------------------
  // THE LAST DATA POINT
  // -----------------------------------------------------------------------------
  let lastDataPoint = data.find(
    (d) => xAccessor(d) === max(data, (d) => xAccessor(d))
  );
  // $: console.log({ lastDataPoint });

  // -----------------------------------------------------------------------------
  // THE ARROW
  // -----------------------------------------------------------------------------
  $: showArrow = Math.abs($nearestDataY - yScale(yAccessor(lastDataPoint))) > 30;

</script>

<g transition:fade={{ duration: transitionDuration, easing: cubicIn }}>
  <!-- last data point - fixed -->

  <!--  horizontal reference -->
  <line
    class="reference-line"
    x1={xScale(xAccessor(lastDataPoint))}
    y1={yScale(yAccessor(lastDataPoint))}
    x2={xScale(1959)}
    y2={yScale(yAccessor(lastDataPoint))}
  />

  <!-- vertical reference -->
  <line
    class="reference-line"
    x1={xScale(xAccessor(lastDataPoint))}
    y1={yScale(yAccessor(lastDataPoint))}
    x2={xScale(xAccessor(lastDataPoint))}
    y2={yScale(0)}
  />

  <rect
    class="reference-text-bg"
    x={xScale(xAccessor(lastDataPoint)) - 25}
    y={yScale(0) + 9}
    height={20}
    width={50}
    fill="white"
  />
  <text
    class="reference-text"
    x={xScale(xAccessor(lastDataPoint))}
    y={yScale(0)}
    dy={25}
    text-anchor="middle">{Math.round(xAccessor(lastDataPoint))}</text
  >

  <circle
    cx={xScale(xAccessor(lastDataPoint))}
    cy={yScale(yAccessor(lastDataPoint))}
    r={5}
    fill="black"
  />
  <!-- nearest data point - moves with cursor -->
  <!-- vertical reference -->
  <line
    class="reference-line"
    x1={$nearestDataX}
    y1={$nearestDataY}
    x2={$nearestDataX}
    y2={yScale(0)}
  />
  <!-- to give text padding -->
  <rect
    class="reference-text-bg"
    x={$nearestDataX - 25}
    y={yScale(0) + 9}
    height={20}
    width={50}
    fill="white"
  />
  <text
    class="reference-text"
    x={$nearestDataX}
    y={yScale(0)}
    dy={25}
    text-anchor="middle">{Math.round(xScale.invert($nearestDataX))}</text
  >
  <!-- horizontal reference -->
  <line
    class="reference-line"
    x1={$nearestDataX}
    y1={$nearestDataY}
    x2={xScale(1959)}
    y2={$nearestDataY}
  />
  <circle cx={$nearestDataX} cy={$nearestDataY} r={5} fill="black" />

  <!-- arrow - showing increase in emissions -->
  <!-- only show if there is space for the arrow head -->
  {#if showArrow}
    <!-- transition need to be repeated within the if to apply -->
    {#key showArrow}
      <g>
        <Arrow
          x1={xScale(2023)}
          y1={$nearestDataY}
          x2={xScale(2023)}
          y2={yScale(yAccessor(lastDataPoint))}
          arrowHeadWidth={10}
          arrowHeadHeight={10}
        />
      </g>
    {/key}
  {/if}

  <!-- try a triangle -->
  <polygon
    points="{$nearestDataX},{$nearestDataY} {xScale(
      xAccessor(lastDataPoint)
    )},{yScale(yAccessor(lastDataPoint))} {xScale(
      xAccessor(lastDataPoint)
    )}, {$nearestDataY}"
    style="opacity:0.5;"
  />
</g>

<style>
  .reference-line {
    stroke: rgb(161, 156, 156);
    stroke-dasharray: 5 3;
    pointer-events: none;
  }

  .reference-text {
    fill: rgb(161, 156, 156);
    pointer-events: none;
  }

  .reference-text-bg {
    pointer-events: none;
  }
</style>
