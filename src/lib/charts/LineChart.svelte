<script>
  // D3
  import { csv } from "d3-fetch";
  import { scaleLinear } from "d3-scale";
  import { extent } from "d3-array";

  // tidy-js
  import { tidy, mutate, filter } from "@tidyjs/tidy";

  // -----------------------------------------------------------------------------
  // IMPORT AND CLEAN DATA FROM
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
      filter((d) => d.year >= 1959),
      mutate({
        // convert strings to numbers
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

  // -----------------------------------------------------------------------------
  // DIMENSIONS
  // -----------------------------------------------------------------------------
  let width = 400;
  let height = 400;

  // -----------------------------------------------------------------------------
  // ACCESSORS
  // -----------------------------------------------------------------------------
  const xAccessor = (d) => d.year;
  const yAccessor = (d) => d.total_ghg_emissions;
  // -----------------------------------------------------------------------------
  // SCALES
  // -----------------------------------------------------------------------------
  $: console.log(data);

  $: xScale = scaleLinear()
    .domain(extent(data, (d) => xAccessor(d)))
    .range([0, width]);

  $: yScale = scaleLinear()
    // .domain(extent(data, (d) => yAccessor(d)))
    .domain([0, 50])
    .range([height, 0]);
</script>

{#await data}
  <p>Loading...</p>
{:then data}
  <svg {width} {height}>
    {#each data as d}
      <circle
        cx={xScale(xAccessor(d))}
        cy={yScale(yAccessor(d))}
        r={1}
        fill="purple"
        stroke="black"
        stroke-width={1}
      />
    {/each}
  </svg>
{/await}

<p>Line Chart</p>
