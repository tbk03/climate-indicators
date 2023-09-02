<script>
  // SVELTE
  import { tweened } from "svelte/motion";
  import { cubicOut, cubicIn } from "svelte/easing";
  import { fade } from "svelte/transition";

  // D3.js
  import { max } from "d3-array";
  import { line, area, curveNatural } from "d3-shape";

  // LOCAL COMPONENTS
  import Arrow from "./Arrow.svelte";
  import { get } from "svelte/store";

  export let positionOnChart;
  export let xScale;
  export let yScale;
  export let xAccessor;
  export let yAccessor;
  export let data;
  export let width;
  export let height;

  console.log({width, height});

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

  console.log(xScale.range()[1]);

  // -----------------------------------------------------------------------------
  // REFERENCE LINES - coordinates in pixels
  // -----------------------------------------------------------------------------
  $: referenceLines = [
    {
      ref: "nearestVert",
      x1: $nearestDataX,
      y1: $nearestDataY,
      x2: $nearestDataX,
      y2: yScale.range()[0] + 5,
    },
    {
      ref: "nearestHorz",
      x1: lastDataPointCoord.x,
      y1: $nearestDataY,
      x2: xScale.range()[0],
      y2: $nearestDataY,
    },
    {
      ref: "lastVert",
      x1: lastDataPointCoord.x,
      y1: lastDataPointCoord.y,
      x2: lastDataPointCoord.x,
      y2: yScale.range()[0] + 5,
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
  let textLabels = [];
  $: {
    textLabels = [
      {
        ref: "x-nearest",
        text: xScale.invert($nearestDataX),
        x: $nearestDataX,
        y: yScale(0),
        dx: 0,
        dy: 21,
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
        dy: 21,
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
  }

  // -----------------------------------------------------------------------------
  // THE ARROW
  // -----------------------------------------------------------------------------
  const minArrowLength = 30; //px
  $: showArrow =
    Math.abs($nearestDataY - lastDataPointCoord.y) > minArrowLength;

  // -----------------------------------------------------------------------------
  // THE ARROW
  // -----------------------------------------------------------------------------

  $: areaGen = area()
    .curve(curveNatural)
    // @ts-ignore
    .x((d) => xScale(xAccessor(d)))
    .y0(yScale(0))
    .y1((d) => yScale(yAccessor(d)))(data);

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

  <!-- Text labels for crosshairs -->
  <g class="text-labels">
    {#each textLabels as tl}
      {#if tl.visible}
        <text
          class="tt-reference-text"
          x={tl.x}
          y={tl.y}
          dx={tl.dx}
          dy={tl.dy}
          text-anchor={tl.textAnchor}
          dominant-baseline={tl.dominantBaseline}
          transition:fade={{ duration: transitionDuration, easing: cubicIn }}
        >
          {Math.round(tl.text)}
        </text>
      {/if}
    {/each}
  </g>

  {#if showArrow}
    <!-- transition need to be repeated within the if to apply -->
    <g transition:fade={{ duration: transitionDuration, easing: cubicIn }}>
      <Arrow
        x1={xScale(2022.2)}
        y1={$nearestDataY}
        x2={xScale(2022.2)}
        y2={yScale(yAccessor(lastDataPoint))}
        arrowHeadWidth={10}
        arrowHeadHeight={10}
      />
    </g>
  {/if}

  <!-- try a triangle -->
  <!-- <polygon
    points="{$nearestDataX},{$nearestDataY} {xScale(
      xAccessor(lastDataPoint)
    )},{yScale(yAccessor(lastDataPoint))} {xScale(
      xAccessor(lastDataPoint)
    )}, {$nearestDataY}"
    style="opacity:0.5;"
  /> -->

  <path
    class="area"
    d={areaGen}
    fill="#C94A54"
    stroke="#C94A54"
    stroke-width={3}
    opacity={1}
    mask="url(#hide-area)"
  />
</g>

<defs>
  <mask id="hide-area">
    <rect x="0" y="0" {width} {height} fill="white" />

    <rect x={0} y={0} width={$nearestDataX} height={height} fill="black" />
    <rect x={0} y={$nearestDataY} width={width} height={height - $nearestDataY} fill="black" />
  </mask>
</defs>

<style>
  .reference-line {
    stroke: rgb(161, 156, 156);
    stroke-dasharray: 5 3;
    pointer-events: none;
  }

  .tt-reference-text {
    pointer-events: none;
  }

  .area {
    transition: d all 1000ms;
  }
</style>
