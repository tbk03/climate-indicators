<script>
  import { make2DArray } from "../util/unit-chart-util.js";

  export let numFilledCells;
  export let year;

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
  // MAKE CHART RESPONSIVE
  // --------------------------------------------------------------
  let chartWidth = 800;
  
  $: {
    unitWidth = (chartWidth / numCols) - gutter;
    unitHeight = unitWidth;

    // console.log({chartWidth, unitWidth});
  }
  // --------------------------------------------------------------
  // UPDATE NUMBER OF UNIT CELLS WHICH ARE FILLED
  // --------------------------------------------------------------

  // EACH TIME THE NUMBER OF FILLED CELLS IS UPDATED
  $: {
    // reset the count of cells that are filled
    let countFilledCells = 0;

    // loop over 2d grid from the bottom row
    for (let i = grid.length - 1; i >= 0; i--) {
      let row = grid[i];
      for (let j = 0; j < row.length; j++) {
        if (countFilledCells < +numFilledCells) {
          grid[i][j] = true;
          countFilledCells++;
        } else {
          grid[i][j] = false;
        }
      }
    }
  }
</script>

<div class="waffle-title chart-title">
  <h1>In {year} {numFilledCells}% GHG budget was left</h1>
</div>
<div class="waffle-container chart-container" >
  <div class="waffle chart" bind:clientWidth={chartWidth}>
    <svg height={chartWidth} width={chartWidth}>
      {#each grid as row, i}
        {#each row as cell, j}
          <rect
            class={grid[i][j] ? "unit filled" : "unit empty"}
            width={unitWidth}
            height={unitHeight}
            x={j * (unitWidth + gutter) + gutter / 2}
            y={i * (unitHeight + gutter) + gutter / 2}
            stroke={grid[i][j] ? "white" : "black"}
            stroke-width={grid[i][j] ? 0 : 1}
            fill={grid[i][j] ? "black" : "white"}
          />
        {/each}
      {/each}
    </svg>
  </div>
</div>

<style>
  /* CONTAINERS */
  .waffle-container {
    

  }

  /* WAFFLE TITLE */
  

  /* WAFFLE DESCRIPTION */

  /* WAFFLE CHART */
  .unit {
    transition-property: fill stroke;
    transition: 1s cubic-bezier(0.5, 1, 0.5, 1);
  }
</style>
