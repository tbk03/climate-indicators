<script>
  import { max } from "d3-array";
  import { fade } from "svelte/transition";

  export let data;
  export let xAccessor;
  export let yAccessor;
  export let xScale;
  export let yScale;
  export let margin;
  export let positionOnChart;
  export let width;
  export let widthThreshold;

  $: lastDataPoint = data.find(
    (d) => xAccessor(d) === max(data, (d) => xAccessor(d))
  );

  $: position = {
    x:
      width > widthThreshold
        ? xScale(xAccessor(lastDataPoint)) + margin.left + 5
        : margin.left + xScale(1963), // choose a point where the tooltip appears of small screens
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

<div
  class="annotation"
  style="top: {position.y}px; left: {position.x}px;"
  transition:fade
>
  <div
    class="percent"
    style="color: {percentageChange > 0 ? '#c94a54' : '#374E83'};"
  >
    <span
      class={`triangle ${percentageChange > 0 ? "up" : "down"}`}
      style="border-color: {percentageChange > 0
        ? 'transparent transparent #c94a54 transparent'
        : 'transparent transparent #374E83 transparent'};"
    />
    <span class="percent-value">{Math.abs(percentageChange.toFixed())}%</span>
  </div>
  <!-- <p>
    {changeDescriptor} between
    <span class="year">{xAccessor(positionOnChart)}</span> &
    <span class="year">{xAccessor(lastDataPoint)}</span>
  </p>  -->
</div>

<style>
  .triangle {
    width: 0px;
    height: 0px;
    border-style: solid;
    border-width: 0 8px 13.9px 8px;

    transform: rotate(0deg);
  }

  .up {
    transform: rotate(0deg);
  }
  .down {
    transform: rotate(180deg);
  }

  .annotation {
    position: absolute;
    color: #2c3130;
    padding: 0 0 0 0;
    margin: 0 30px 0 5px;
    font-size: 16px;
    text-align: center;
    z-index: 0;
    pointer-events: none;
  }

  div.percent {
    width: 110px;
    display: flex;
    align-items: center;
    justify-content: space-around;
    margin: 0;
    padding: 15px 10px 15px 10px;
    border-radius: 5px;
    background: rgba(209, 148, 153, 20%);
    background: linear-gradient(
      164deg,
      rgba(247, 238, 239, 100%) 0%,
      rgba(252, 249, 250, 100%) 100%
    );
    font-weight: 700;
    font-size: 30px;
    box-shadow: 2px 3px 3px rgba(209, 148, 153, 50%);
  }
</style>
