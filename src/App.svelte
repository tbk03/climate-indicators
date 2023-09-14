<script>
  // import WaffleChart from "./lib/charts/Waffle-Chart.svelte";
  // import YearSlider from "./lib/components/YearSlider.svelte";
  // import SlopeChart from "./lib/charts/Slope-Chart.svelte";
  import LineChart from "./lib/charts/LineChart.svelte";

  // DATA WRANGLING
  import { tidy, mutate, filter } from "@tidyjs/tidy";
  // import { extent, max, min } from "d3-array";
  import { csv } from "d3-fetch";

  // --------------------------------------------------------------
  // UPDATE NUMBER OF UNIT CELLS WHICH ARE FILLED
  // --------------------------------------------------------------
  // let percGHGBudget = 50; // initial number of cells to fill
  // $: year = percGHGBudget + 1900;

  // -----------------------------------------------------------------------------
  // IMPORT AND CLEAN DATA FROM GITHUB
  // -----------------------------------------------------------------------------

  let data = [];
  loadData();

  // LOAD FROM GITHUB
  async function loadData() {
    const response = await csv(
      "https://raw.githubusercontent.com/ClimateIndicator/GHG-Emissions-Assessment/main/results/ghg_emissions_co2e.csv"
    );
    data = await processData(response);
  }

  // CLEAN DATA USING tidy.js
  async function processData(data) {
    const processed = tidy(
      data,
      // data quality poor before 1959
      // @ts-ignore
      filter((d) => d.year >= 1959),
      mutate({
        // convert strings to numbers
        // @ts-ignore
        year: (d) => +d.year,
        "F-gases": (d) => +d["F-gases"],
        N2O: (d) => +d["N2O"],
        CH4: (d) => +d["CH4"],
        "CO2-LULUCF": (d) => +d["CO2-LULUCF"],
        "CO2-FFI": (d) => +d["CO2-FFI"],

        // calculate total emmission each year
        total_ghg_emissions: (d) =>
          d["F-gases"] + d["N2O"] + d["CH4"] + d["CO2-LULUCF"] + d["CO2-FFI"],
      })
    );
    return processed;
  }
</script>

<main>
  <section class="interative-section">
    <!-- <YearSlider bind:sliderValue={percGHGBudget} />
    <WaffleChart numFilledCells={percGHGBudget} {year} />
    <SlopeChart {year} /> -->
    <div>hello</div>
    <LineChart {data}/>
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
