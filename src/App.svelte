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
  // start from bottom of array

</script>

<main>
  <div class="range-container">
    <input type="range" min="0" max="100" bind:value={numCells} />
  </div>
  <svg height="400" width="400">
    {#each grid as row, i}
      {#each row as cell, j}
        {#if grid[i][j]}
          <rect width="10" height="10" x={j * 11} y={i * 11} />
        {/if}
      {/each}
    {/each}
  </svg>
</main>

<style>
</style>
