<script>
  import { make2DArray } from "./lib/unit-chart-util.js";
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
  let numCells = 50; // initial number of cells to fill

  // EACH TIME THE NUMBER OF FILLED CELLS IS UPDATED
  $: {
    // reset the count of cells that are filled
    let countFilledCells = 0;

    // loop over 2d grid from the bottom row
    for (let i = grid.length - 1; i >= 0; i--) {
      let row = grid[i];
      for (let j = 0; j < row.length; j++) {
        if (countFilledCells < +numCells) {
          grid[i][j] = true;
          countFilledCells++;
        } else {
          grid[i][j] = false;
        }
      }
    }
  }
</script>

<main>
  <div class="range-container">
    <input type="range" min="0" max="100" bind:value={numCells} />
  </div>
  <svg height="400" width="400">
    {#each grid as row, i}
      {#each row as cell, j}
        <!-- {#if grid[i][j]} -->
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
        <!-- {/if} -->
        <!-- reference grid -->
        <!-- <rect
          width={unitWidth + gutter}
          height={unitHeight + gutter}
          x={j * (unitWidth + gutter)}
          y={i * (unitHeight + gutter)}
          fill="none"
          stroke="black"
        /> -->
      {/each}
    {/each}
  </svg>
</main>

<style>
  /* .unit.empty {
    fill: white;
  }
  .unit.filled {
    fill: black;
  } */
  .unit {
    transition-property: fill stroke;
    transition: 1s cubic-bezier(0.5, 1, 0.5, 1);
  }
</style>
