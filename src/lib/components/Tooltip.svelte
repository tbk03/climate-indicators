<script>
  import { tweened } from "svelte/motion";
  import { cubicOut } from 'svelte/easing';
  export let positionOnChart;
  export let xScale;
  export let yScale;
  export let xAccessor;
  export let yAccessor;
  export let data;

  let dataPoint = data.find((d) => xAccessor(d) === xAccessor(positionOnChart));

  const tweenedTransition = {
    duration: 200,
    easing: cubicOut
  }
  let nearestDataX = tweened(xAccessor(dataPoint), tweenedTransition);
  let nearestDataY = tweened(yAccessor(dataPoint), tweenedTransition);

  $: {
    dataPoint = data.find((d) => xAccessor(d) === xAccessor(positionOnChart));
    nearestDataX.set(xAccessor(dataPoint));
    nearestDataY.set(yAccessor(dataPoint));
  }
  
  //   $: console.log(dataPoint);
  //   $: nearestDataX = tweened();
</script>

<circle
  cx={xScale($nearestDataX)}
  cy={yScale($nearestDataY)}
  r={10}
  fill="red"
/>
