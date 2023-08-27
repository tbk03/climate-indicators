<!-- Code adapted from Better Data Visualisations with Svelte by Connor Rothschild  -->
<script>
  export let yScale;
  export let xScale;
  export let width;
  $: yTicks = yScale.ticks(5);
</script>

<g class="axis-y">
  {#each yTicks as tick, index}
    <g class="tick" transform="translate(0, {yScale(tick)})">
      <line
        x1={index === yTicks.length - 1
          ? xScale(1971)
          : index === 0
          ? 0
          : xScale(1960)}
        x2={width}
        y1={0}
        y2={0}
        stroke={index === 0 ? "#8f8f8f" : "#e5e7eb"}
      />
      <text
        class="axis-tick-label"
        x="-20"
        y="0"
        dominant-baseline="middle"
        >{tick} {index === yTicks.length - 1 ? " Gt CO2e" : ""}</text
      >
    </g>
  {/each}
</g>
