<script>
  import { make2DArray } from "./lib/unit-chart-util.js";
  import WaffleChart from "./lib/Waffle-Chart.svelte";
  // --------------------------------------------------------------
  // SET UP UNIT CHART
  // --------------------------------------------------------------

  // DIMENSIONS
  let numRows = 10;
  let numCols = 10;
  let unitHeight = 20;
  let unitWidth = 20;
  let gutter = 6;

  // DATA STRUCTURE
  let grid = make2DArray(numRows, numCols);

  // --------------------------------------------------------------
  // UPDATE NUMBER OF UNIT CELLS WHICH ARE FILLED
  // --------------------------------------------------------------
  let numFilledCells = 50; // initial number of cells to fill
</script>

<main>
  <div class="range-container">
    <input type="range" min="0" max="100" bind:value={numFilledCells} />
  </div>
  <svg height="400" width="400">
    <WaffleChart {grid} {unitHeight} {unitWidth} {gutter} {numFilledCells} />
  </svg>
</main>

<style>
</style>
