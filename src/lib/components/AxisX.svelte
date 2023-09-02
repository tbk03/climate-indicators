<!-- Code adapted from Better Data Visualisations with Svelte by Connor Rothschild  -->
<script>

  import { cubicOut } from "svelte/easing";
  import { fade } from "svelte/transition";

  export let xScale;
  export let height;
  export let width;

  $: xTicks = xScale.ticks(5);

  const transitionDuration = 1000;
  const transition = {
    duration: transitionDuration,
    easing: cubicOut,
  };
</script>

<g class="axis x" transform="translate(0, {height})" transition:fade={transition}>
  {#each xTicks as tick, index}
    <g class="tick" transform="translate({xScale(tick)}, 0)" >
      <line x1="0" x2="0" y1="0" y2="6" stroke="#757373" />
      <text
        class="axis-tick-label"
        x={0}
        y={7}
        dy={10}
        dominant-baseline="middle"
        text-anchor="middle">{tick}</text
      >
    </g>
  {/each}
</g>
