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
  <text class="axis-label">Gt CO2e</text>
  {#each yTicks as tick, index}
    <g class="tick" transform="translate(0, {yScale(tick)})">
      <line
        x1={xScale(1959)}
        x2={width}
        y1={0}
        y2={0}
        opacity={hoveredEvent ? 0.4 : 1}
        stroke={index === 0 ? "#8f8f8f" : "#e5e7eb"}
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
  .y-grid{
    transition-property: opacity;
    transition-duration: 1500ms;
    transition-timing-function: cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }
</style>
