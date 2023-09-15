<script>
  export let data;

  // data wrangling
  import { tidy, mutate, filter, lag, mutateWithSummary } from "@tidyjs/tidy";
  import { max, min } from "d3-array";

  // encoding data to visual parameters
  import { scaleLinear } from "d3-scale";
  import { scale } from "svelte/transition";

  // SVELTE
  import { tweened } from "svelte/motion";
  import { cubicOut, cubicIn } from "svelte/easing";
  import { fade } from "svelte/transition";

  $: console.log({ data });

  // -----------------------------------------------------------------------------
  // DATA WRANGLING
  // -----------------------------------------------------------------------------
  let processedData;
  let years_ago_1 = 10;
  // let years_ago_2 = 5;

  $: {
    processedData = tidy(
      data,

      // find data from n years_ago
      mutateWithSummary({
        total_ghg_emissions_years_ago_1: lag("total_ghg_emissions", {
          n: years_ago_1,
        }),
        // total_ghg_emissions_years_ago_2: lag("total_ghg_emissions", {
        //   n: years_ago_2,
        // }),
      }),

      // comparisions can't be made where data does not go far back enough
      filter((d) => d.total_ghg_emissions_years_ago_1 !== undefined),
      // filter((d) => d.total_ghg_emissions_years_ago_2 !== undefined),

      // calculate change in emissions based on selected year
      mutate({
        change_em_years_ago_1: (d) =>
          // @ts-ignore
          d.total_ghg_emissions - d.total_ghg_emissions_years_ago_1,
        // change_em_years_ago_2: (d) =>
        //   // @ts-ignore
        //   d.total_ghg_emissions - d.total_ghg_emissions_years_ago_2,
      })
    );
  }

  $: console.log({ processedData });

  // -----------------------------------------------------------------------------
  // SELECTED YEARS
  // -----------------------------------------------------------------------------
  let year1 = 2019;
  // const year2 = 1981;

  $: selectedData = processedData.filter(
    (d) => d.year === year1 //|| d.year === year2
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
  // let year1OuterDim;

  const totalAccessor = (d) => (d ? d.total_ghg_emissions : undefined);
  const growthAccessor = (d) => (d ? d.change_em_years_ago_1 : undefined);

  const transitionDuration = 200;
  const tweenedTransition = {
    duration: transitionDuration,
    easing: cubicOut,
  };

  // tweened coordinates in pixels
  let year1OuterDim = tweened(year1, tweenedTransition);
  let year1InnerDim = tweened(year1, tweenedTransition);

  let emissionsYear1, growthYear1;

  $: if (data.length > 0) {
    // OUTER SQUARE - TOTAL EMISSIONS
    emissionsYear1 = totalAccessor(selectedData.find((d) => d.year == year1));
    year1OuterDim.set(Math.round(Math.sqrt(areaScale(emissionsYear1))));

    // OUTER SQUARE - GROWTH IN EMISSIONS
    growthYear1 = growthAccessor(selectedData.find((d) => d.year == year1));
    year1InnerDim.set(Math.round(Math.sqrt(areaScale(growthYear1))));

    console.log({ year1InnerDim });
  }

  // -----------------------------------------------------------------------------
  // INTERACTION
  // -----------------------------------------------------------------------------

  // let sliderYear;
  // $: console.log({sliderYear});

</script>

<div>Treemap</div>
<div>
  <input
    type="range"
    id="year"
    name="year"
    min={min(processedData, (d) => d.year)}
    max={max(processedData, (d) => d.year)}
    step="1"
    bind:value={year1}
  />
  <label for="year">Year</label>
  <div>{year1}</div>
</div>

<svg width="600" height="400">
  <rect x="0" y="0" width={$year1OuterDim} height={$year1OuterDim} />
  <rect x="0" y="0" width={$year1InnerDim} height={$year1InnerDim} fill="white" />
</svg>
