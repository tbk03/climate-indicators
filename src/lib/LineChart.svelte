<script>
  import { csv } from "d3-fetch";
  import { tidy, mutate, filter } from "@tidyjs/tidy";

  async function processData(data) {
    const processed = tidy(
      data,
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

  async function loadData() {
    const response = await csv(
      "https://raw.githubusercontent.com/ClimateIndicator/GHG-Emissions-Assessment/main/results/ghg_emissions_co2e.csv"
    );
    const processed = await processData(response);
    console.log(processed);
    return processed;
  }

  let data = loadData();
  
</script>

<p>Line Chart</p>
