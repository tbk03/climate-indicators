<script>
  import { max } from "d3-array";

  export let data;
  export let xAccessor;
  export let yAccessor;
  export let xScale;
  export let yScale;
  export let margin;
  export let positionOnChart;

  $: lastDataPoint = data.find(
    (d) => xAccessor(d) === max(data, (d) => xAccessor(d))
  );

  $: position = {
    x: xScale(xAccessor(lastDataPoint)) + margin.left + 30,
    y: yScale(yAccessor(lastDataPoint)) + margin.top,
  };

  let percentageChange;
  $: {
    let selectedGHGEmissions = yAccessor(positionOnChart);
    let lastGHGEmissions = yAccessor(lastDataPoint);

    percentageChange =
      ((lastGHGEmissions - selectedGHGEmissions) / selectedGHGEmissions) * 100;
  }

  let changeDescriptor;
  $: {
    if (percentageChange > 0) {
      changeDescriptor = "increase";
    } else if (percentageChange < 0) {
      changeDescriptor = "decrease";
    } else {
      changeDescriptor = "change";
    }
  }
</script>

<div class="annotation" style="top: {position.y}px; left: {position.x}px;">
  <span>{Math.abs(percentageChange.toFixed())}</span>%
  <p>
    {changeDescriptor} in emissions between {xAccessor(positionOnChart)} and {xAccessor(
      lastDataPoint
    )}
  </p>
</div>

<style>
  .annotation {
    position: absolute;
  }
</style>
