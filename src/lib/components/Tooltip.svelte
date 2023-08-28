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
  console.log({ data });

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
  $: console.log({ lastDataPoint });
</script>

<g transition:fade={{ duration: transitionDuration, easing: cubicIn }}>
  <!-- last data point - fixed -->

  <line
    x1={xScale(xAccessor(lastDataPoint))}
    y1={yScale(yAccessor(lastDataPoint))}
    x2={xScale(1958)}
    y2={yScale(yAccessor(lastDataPoint))}
    stroke="black"
  />
  <line
    x1={xScale(xAccessor(lastDataPoint))}
    y1={yScale(yAccessor(lastDataPoint))}
    x2={xScale(xAccessor(lastDataPoint))}
    y2={yScale(0)}
    stroke="black"
  />
  <circle
    cx={xScale(xAccessor(lastDataPoint))}
    cy={yScale(yAccessor(lastDataPoint))}
    r={5}
    fill="black"
  />
  <!-- nearest data point - moves with cursor -->
  <line
    x1={$nearestDataX}
    y1={$nearestDataY}
    x2={$nearestDataX}
    y2={yScale(0)}
    stroke="black"
  />
  <line
    x1={$nearestDataX}
    y1={$nearestDataY}
    x2={xScale(1958)}
    y2={$nearestDataY}
    stroke="black"
  />
  <circle cx={$nearestDataX} cy={$nearestDataY} r={5} fill="black" />

  <!-- arrow - showing increase in emissions -->
  <!-- only show if there is space for the arrow head -->
  {#if Math.abs($nearestDataY - yScale(yAccessor(lastDataPoint))) > 30}
  <!-- transition need to be repeated within the if to apply -->
  <g transition:fade={{ duration: transitionDuration, easing: cubicIn }}>
      <Arrow
        x1={xScale(2023)}
        y1={$nearestDataY}
        x2={xScale(2023)}
        y2={yScale(yAccessor(lastDataPoint))}
        arrowHeadWidth={10}
        arrowHeadHeight={10}
      />
    </g>
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
