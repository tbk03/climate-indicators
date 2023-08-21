<script>
  import WaffleChart from "./lib/Waffle-Chart.svelte";
  import YearSlider from "./lib/YearSlider.svelte";


  // --------------------------------------------------------------
  // UPDATE NUMBER OF UNIT CELLS WHICH ARE FILLED
  // --------------------------------------------------------------
  let percGHGBudget = 50; // initial number of cells to fill
</script>

<main>
    <YearSlider bind:sliderValue={percGHGBudget}/>
    <WaffleChart numFilledCells={percGHGBudget} />
</main>


