<script>
  export let data;

  // data wrangling
  import { tidy, mutate, filter, lag, mutateWithSummary } from "@tidyjs/tidy";
  import { max, min } from "d3-array";

  // encoding data to visual parameters
  import { scaleLinear } from "d3-scale";
  import { scale } from "svelte/transition";

  //   $: console.log(data);

  // -----------------------------------------------------------------------------
  // DATA WRANGLING
  // -----------------------------------------------------------------------------
  let processedData;
  let years_ago_1 = 10;
  let years_ago_2 = 5;

  $: {
    processedData = tidy(
      data,

      // find data from n years_ago
      mutateWithSummary({
        total_ghg_emissions_years_ago_1: lag("total_ghg_emissions", {
          n: years_ago_1,
        }),
        total_ghg_emissions_years_ago_2: lag("total_ghg_emissions", {
          n: years_ago_2,
        }),
      }),

      // comparisions can't be made where data does not go far back enough
      filter((d) => d.total_ghg_emissions_years_ago_1 !== undefined),
      filter((d) => d.total_ghg_emissions_years_ago_2 !== undefined),

      // calculate change in emissions based on selected year
      mutate({
        change_em_years_ago_1: (d) =>
          // @ts-ignore
          d.total_ghg_emissions - d.total_ghg_emissions_years_ago_1,
        change_em_years_ago_2: (d) =>
          // @ts-ignore
          d.total_ghg_emissions - d.total_ghg_emissions_years_ago_2,
      })
    );
  }

  $: console.log({ processedData });

  // -----------------------------------------------------------------------------
  // SELECTED YEARS
  // -----------------------------------------------------------------------------
  const year1 = 2019;
  const year2 = 1981;

  $: selectedData = processedData.filter(
    (d) => d.year === year1 || d.year === year2
  );

  $: console.log({ selectedData });

  // -----------------------------------------------------------------------------
  // SCALES
  // -----------------------------------------------------------------------------
  const maxRectWidth = 400;
  const maxRectHeight = 400;
  $: maxEmissons = max(processedData, (d) => d.total_ghg_emissions);

  $: areaScale = scaleLinear()
    .domain([0, maxEmissons])
    .range([0, maxRectWidth * maxRectHeight]);

  // -----------------------------------------------------------------------------
  // RECTANGLE DIMENSIONS
  // -----------------------------------------------------------------------------
  const heightRect1Year1 = 400;
  const areaAccessor = (d) => d.total_ghg_emissions;

  $: emissionsYear1 = areaAccessor(selectedData.find(d => d.year == year1));
  $: widthRect1Year1 = areaScale(emissionsYear1) / heightRect1Year1;
  $: console.log({widthRect1Year1});
</script>

<div>Treemap</div>
<svg width="600" height="400">
  <rect x="0" y="0" width={widthRect1Year1} height={heightRect1Year1} />
</svg>
