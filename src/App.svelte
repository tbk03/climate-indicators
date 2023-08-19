<script>
  function make2DArray(cols, rows) {
    let arr = new Array(cols);
    for (let i = 0; i < arr.length; i++) {
      arr[i] = new Array(rows);
    }
    return arr;
  }

  let grid = make2DArray(10, 10);

  let numCells = 50;
  let cellsVisible = 0;

  $: {
    // start from bottom of array
    for (let i = grid.length - 1; i >= 0; i--) {
      let row = grid[i];
      for (let j = 0; j < row.length; j++) {
        if (cellsVisible < +numCells) {
          grid[i][j] = true;
          cellsVisible++;
        } else {
          grid[i][j] = false;
        }
      }
    }
    cellsVisible = 0;
  }

  let unitHeight = 20;
  let unitWidth = 20;
  let gutter = 6;
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
  .unit{
    transition-property: fill stroke;
    transition: 1s cubic-bezier(0.5, 1, 0.5, 1);
  }
</style>
