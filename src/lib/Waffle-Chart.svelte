<script>
  export let grid;
  export let unitWidth;
  export let unitHeight;
  export let gutter;
  export let numFilledCells;

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

<div class="waffle-container">
  <div class="waffle-desc">
    <h2>In 19..</h2>
    <!-- {#key numFilledCells} -->
        <h1 class="waffle-percent">{numFilledCells}%</h1>
    <!-- {/key} -->
    <h2>Of the GHG budget remained</h2>

  </div>

  <div class="waffle">
    <svg height="400" width="400">
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
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
  }

  /* WAFFLE DESCRIPTION */
  .waffle-desc {
    max-width: 100px;
  }

  /* WAFFLE CHART */
  .waffle {
    /* flex-grow: 1; */
  }
  .unit {
    transition-property: fill stroke;
    transition: 1s cubic-bezier(0.5, 1, 0.5, 1);
  }
</style>
