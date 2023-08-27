<!-- Code adapted from Better Data Visualisations with Svelte by Connor Rothschild  -->
<script>
  export let yScale;
  export let xScale;
  export let width;
  $: yTicks = yScale.ticks(5);
</script>

<!-- filter for solid svg text background -->
<defs>
  <filter x="0" y="0" width="1" height="1" id="solid">
    <feFlood flood-color="white" result="bg" />
    <feMerge>
      <feMergeNode in="bg" />
      <feMergeNode in="SourceGraphic" />
    </feMerge>
  </filter>
</defs>

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
        filter="url(#solid)"
        >{tick} {index === yTicks.length - 1 ? " Gt CO2e" : ""}</text
      >
    </g>
  {/each}
</g>
