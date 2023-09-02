<!-- Code adapted from Better Data Visualisations with Svelte by Connor Rothschild  -->
<script>
  import { cubicOut } from "svelte/easing";
  import { fade } from "svelte/transition";

  export let yScale;
  export let xScale;
  export let width;
  export let hoveredEvent;

  const transitionDuration = 1000;
  const transition = {
    duration: transitionDuration,
    easing: cubicOut,
  };

  $: yTicks = yScale.ticks(5);
</script>

<g class="axis-y">
  <text class="axis-label" y="-20">
    <tspan class="axis-label-variable">GLOBAL CARBON EMISSIONS</tspan>
  </text>
  <text y="0">
    <tspan class="units">Gt CO</tspan><tspan dy="0.2em" font-size=".6em"
      >2</tspan
    ><tspan dy="-0.2em" font-size="1em">e</tspan></text
  >
  {#each yTicks as tick, index}
    <g class="tick" transform="translate(0, {yScale(tick)})">
      <line
        x1={xScale(1959)}
        x2={width}
        y1={0}
        y2={0}
        opacity={hoveredEvent ? 0.4 : 1}
        stroke={index === 0 ? "#757373" : "#E7E5E4"}
        class="y-grid"
      />
      {#if !hoveredEvent}
        <text
          class="axis-tick-label"
          x="-20"
          y="0"
          dominant-baseline="middle"
          transition:fade={transition}
          >{tick}
        </text>
      {/if}
    </g>
  {/each}
</g>

<style>
  .y-grid {
    transition-property: opacity;
    transition-duration: 1500ms;
    transition-timing-function: cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }

  .axis-label-variable {
    font-weight: 700;
    font-size: 14px;
  }

  .units {
    font-weight: 400;
    font-size: 14px;
  }
</style>
