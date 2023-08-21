<script>
  import WaffleChart from "./lib/Waffle-Chart.svelte";
  import YearSlider from "./lib/YearSlider.svelte";
  import SlopeChart from "./lib/Slope-Chart.svelte";

  // --------------------------------------------------------------
  // UPDATE NUMBER OF UNIT CELLS WHICH ARE FILLED
  // --------------------------------------------------------------
  let percGHGBudget = 50; // initial number of cells to fill

  $: year = percGHGBudget + 1900;
</script>

<main>
  <section class="interative-section">
    <YearSlider bind:sliderValue={percGHGBudget} />
    <WaffleChart numFilledCells={percGHGBudget} {year} />
    <SlopeChart {year} />
  </section>
</main>

<style>
  /* COMMON STYLING FOR CHARTS */
  :global(.chart-title) {
    text-align: center;
  }

  :global(.chart) {
    margin: 0 auto;
    max-width: 600px;
  }
</style>
