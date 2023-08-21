<script>
  import WaffleChart from "./lib/Waffle-Chart.svelte";


  // --------------------------------------------------------------
  // UPDATE NUMBER OF UNIT CELLS WHICH ARE FILLED
  // --------------------------------------------------------------
  let percGHGBudget = 50; // initial number of cells to fill
</script>

<main>
  <div class="range-container">
    <input type="range" min="0" max="100" bind:value={percGHGBudget} />
  </div>
    <WaffleChart numFilledCells={percGHGBudget} />
</main>

<style>
</style>
