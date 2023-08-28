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

  // tweened coordinates in pixels
  let nearestDataX = tweened(xScale(xAccessor(dataPoint)), tweenedTransition);
  let nearestDataY = tweened(yScale(yAccessor(dataPoint)), tweenedTransition);

  $: {
    dataPoint = data.find((d) => xAccessor(d) === xAccessor(positionOnChart));
    nearestDataX.set(xScale(xAccessor(dataPoint)));
    nearestDataY.set(yScale(yAccessor(dataPoint)));
  }

  // -----------------------------------------------------------------------------
  // THE LAST DATA POINT
  // -----------------------------------------------------------------------------
  const lastDataPoint = data.find(
    (d) => xAccessor(d) === max(data, (d) => xAccessor(d))
  );

  const lastDataPointCoord = {
    x: xScale(xAccessor(lastDataPoint)),
    y: yScale(yAccessor(lastDataPoint)),
  };

  console.log({ lastDataPointCoord });

  // -----------------------------------------------------------------------------
  // REFERENCE LINES - coordinates in pixels
  // -----------------------------------------------------------------------------
  $: referenceLines = [
    {
      ref: "nearestVert",
      x1: $nearestDataX,
      y1: $nearestDataY,
      x2: $nearestDataX,
      y2: yScale.range()[0],
    },
    {
      ref: "nearestHorz",
      x1: $nearestDataX,
      y1: $nearestDataY,
      x2: xScale.range()[0],
      y2: $nearestDataY,
    },
    {
      ref: "lastVert",
      x1: lastDataPointCoord.x,
      y1: lastDataPointCoord.y,
      x2: lastDataPointCoord.x,
      y2: yScale.range()[0],
    },
    {
      ref: "lastHorz",
      x1: lastDataPointCoord.x,
      y1: lastDataPointCoord.y,
      x2: xScale.range()[0],
      y2: lastDataPointCoord.y,
    },
  ];

  // -----------------------------------------------------------------------------
  // CIRCLES - coordinates in pixels
  // -----------------------------------------------------------------------------
  $: circles = [
    { ref: "nearest", cx: $nearestDataX, cy: $nearestDataY },
    { ref: "last", cx: lastDataPointCoord.x, cy: lastDataPointCoord.y },
  ];

  // -----------------------------------------------------------------------------
  // TEXT LABELS - coordinates in pixels
  // -----------------------------------------------------------------------------
  $: textLabels = [
    {
      ref: "x-nearest",
      text: xScale.invert($nearestDataX),
      x: $nearestDataX,
      y: yScale(0),
      dx: 0,
      dy: 25,
      textAnchor: "middle",
      dominantBaseline: "auto",
      visible: true,
    },
    {
      ref: "x-last",
      text: xScale.invert(lastDataPointCoord.x),
      x: lastDataPointCoord.x,
      y: yScale(0),
      dx: 0,
      dy: 25,
      textAnchor: "middle",
      dominantBaseline: "auto",
      visible:
        Math.abs(lastDataPointCoord.x - $nearestDataX) < 30 ? false : true,
    },
    {
      ref: "y-nearest",
      text: yScale.invert($nearestDataY),
      x: xScale.range()[0],
      y: $nearestDataY,
      dx: -5,
      dy: 0,
      textAnchor: "end",
      dominantBaseline: "middle",
      visible: true,
    },
    {
      ref: "y-last",
      text: yScale.invert(lastDataPointCoord.y),
      x: xScale.range()[0],
      y: lastDataPointCoord.y,
      dx: -5,
      dy: 0,
      textAnchor: "end",
      dominantBaseline: "middle",
      visible:
        Math.abs(lastDataPointCoord.y - $nearestDataY) < 20 ? false : true,
    },
  ];

  $: console.log(xScale(0));
  // -----------------------------------------------------------------------------
  // THE ARROW
  // -----------------------------------------------------------------------------
  const minArrowLength = 30; //px
  $: showArrow =
    Math.abs($nearestDataY - lastDataPointCoord.y) > minArrowLength;
</script>

<!-- TOOLTIP GROUP -->
<g transition:fade={{ duration: transitionDuration, easing: cubicIn }}>
  <!-- REFERENCE LINES -->
  <g class="reference-lines">
    {#each referenceLines as rl}
      <line
        class="reference-line"
        x1={rl.x1}
        y1={rl.y1}
        x2={rl.x2}
        y2={rl.y2}
      />
    {/each}
  </g>

  <!-- CIRCLES -->
  <g class="circles">
    {#each circles as c}
      <circle cx={c.cx} cy={c.cy} r={5} fill="black" />
    {/each}
  </g>

  <!-- last data point - fixed -->

  <g class="text-labels">
    {#each textLabels as tl}
      <text
        class="reference-text"
        x={tl.x}
        y={tl.y}
        dx={tl.dx}
        dy={tl.dy}
        text-anchor={tl.textAnchor}
        dominant-baseline={tl.dominantBaseline}
      >
        {#if tl.visible}
          {#key tl.visible}
            {Math.round(tl.text)}
          {/key}
        {/if}
      </text>
    {/each}
  </g>

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
</style>
