<script>
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";
  export let positionOnChart;
  export let xScale;
  export let yScale;
  export let xAccessor;
  export let yAccessor;
  export let data;

  let dataPoint; 
  dataPoint = data.find((d) => xAccessor(d) === xAccessor(positionOnChart));
  //   console.log({ dataPoint });

  const tweenedTransition = {
    duration: 200,
    easing: cubicOut,
  };

  let nearestDataX;
  let nearestDataY;
  if (data) {
    nearestDataX = tweened(xAccessor(dataPoint), tweenedTransition);
    nearestDataY = tweened(yAccessor(dataPoint), tweenedTransition);
  }

  $: {
    dataPoint = data.find((d) => xAccessor(d) === xAccessor(positionOnChart));
    nearestDataX.set(xAccessor(dataPoint));
    nearestDataY.set(yAccessor(dataPoint));
  }

  //   $: console.log(dataPoint);
  //   $: nearestDataX = tweened();
</script>

{#await data then data}
  <circle
    cx={xScale($nearestDataX)}
    cy={yScale($nearestDataY)}
    r={10}
    fill="red"
  />
{/await}
