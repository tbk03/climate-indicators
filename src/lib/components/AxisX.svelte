<!-- Code adapted from Better Data Visualisations with Svelte by Connor Rothschild  -->
<script>
  export let xScale;
  export let height;
  export let width;

  $: xTicks = xScale.ticks(5);
  $: console.log({ xTicks });
</script>

<g class="axis x" transform="translate(0, {height})">
  {#each xTicks as tick, index}
    <g class="tick" transform="translate({xScale(tick)}, 0)">
      <line x1="0" x2="0" y1="0" y2="6" stroke="hsla(212, 10%, 53%, 1" />
      <text
        class="axis-tick-label"
        x={0}
        y={6}
        dy={10}
        dominant-baseline="middle"
        text-anchor={index === 0 ? "start" : "middle"}>{tick}</text
      >
    </g>
  {/each}
</g>
