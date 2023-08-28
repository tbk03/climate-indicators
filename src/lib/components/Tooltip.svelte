<script>
  // SVELTE
  import { tweened } from "svelte/motion";
  import { cubicOut, cubicIn } from "svelte/easing";
  import {fade} from "svelte/transition";

  // D3.js
  import { max } from "d3-array";

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

<g transition:fade={{duration: transitionDuration, easing: cubicIn}}>
  <!-- fixed on last data point -->
  <circle
    cx={xScale(xAccessor(lastDataPoint))}
    cy={yScale(yAccessor(lastDataPoint))}
    r={10}
    fill="red"
  />
  <!-- moves with cursor -->
  <circle cx={$nearestDataX} cy={$nearestDataY} r={10} fill="red" />
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
    x2={xScale(2030)}
    y2={$nearestDataY}
    stroke="black"
  />
</g>
