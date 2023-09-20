<script>
  export let data;

  // data wrangling
  import {
    tidy,
    mutate,
    filter,
    lag,
    mutateWithSummary,
    addRows,
  } from "@tidyjs/tidy";
  import { max, min } from "d3-array";
  import { stratify, treemap, treemapBinary, hierarchy } from "d3-hierarchy";

  // encoding data to visual parameters
  import { scaleLinear } from "d3-scale";
  import { scale } from "svelte/transition";

  // SVELTE
  import { tweened } from "svelte/motion";
  import { cubicOut, cubicIn } from "svelte/easing";
  import { fade } from "svelte/transition";

  // $: console.log({ data });

  // -----------------------------------------------------------------------------
  // DATA CLEANING
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

      mutate({
        // calculate change in emissions based on selected year
        change_em_years_ago_1: (d) =>
          // @ts-ignore
          d.total_ghg_emissions - d.total_ghg_emissions_years_ago_1,

        // required for creating tree data structure
        decade: (d) => `${d.year - (d.year % 10)}`,
        parent: (d) => "whole",
      })
    );
  }

  $: console.log({ processedData });

  // -----------------------------------------------------------------------------
  // FORMAT DATA FOR D3 TREEMAP
  // -----------------------------------------------------------------------------

  let stratifyData = stratify()
    .id((d) => d.decade)
    .parentId((d) => d.parent);

  let processedDataTree;
  $: stratifyInput = tidy(
    processedData,
    // @ts-ignore
    filter((d) => d.year % 10 === 0),
    addRows({ decade: "whole", parent: "" })
  );

  $: if (stratifyData) {
    processedDataTree = stratifyData(stratifyInput).sum((d) => d.total_ghg_emissions);
  }
  $: console.log({ processedDataTree });


  // -----------------------------------------------------------------------------
  // CALCULATE LAYOUT FOR D3 TREEMAP
  // -----------------------------------------------------------------------------
  // see https://tonydang.blog/d3-svelte-treemap/ for more details
  $: layout = treemap()
    .tile(treemapBinary)
    .size([400, 400])
    .padding(1)
    .round(true)(processedDataTree);

  $: console.log(processedDataTree);

  // $: dataHierarchy = hierarchy(processedDataTree)
  //     .sum((d) => d.total_ghg_emissions)
  //     .sort((a, b) => b.total_ghg_emissions - a.total_ghg_emissions)
  // $: layout = root(dataHierarchy);

  // $: console.log({processedDataTree});

  // -----------------------------------------------------------------------------
  // SELECTED YEARS
  // -----------------------------------------------------------------------------
  let year1 = 2019;
  // const year2 = 1981;

  $: selectedData = processedData.filter(
    (d) => d.year === year1 //|| d.year === year2
  );

  // $: console.log({ selectedData });

  // -----------------------------------------------------------------------------
  // SCALES
  // -----------------------------------------------------------------------------
  const maxRectWidth = 200;
  const maxRectHeight = 200;
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

  const calcSquareDim = (d) => Math.round(Math.sqrt(areaScale(d)));

  const transitionDuration = 500;
  const tweenedTransition = {
    duration: transitionDuration,
    easing: cubicOut,
  };

  // tweened coordinates in pixels
  let year1OuterDim = tweened(maxRectWidth, tweenedTransition);
  let year1InnerDim = tweened(maxRectWidth, tweenedTransition);

  let emissionsYear1, growthYear1;

  $: if (data.length > 0) {
    // OUTER SQUARE - TOTAL EMISSIONS
    emissionsYear1 = totalAccessor(selectedData.find((d) => d.year == year1));
    year1OuterDim.set(calcSquareDim(emissionsYear1));

    // OUTER SQUARE - GROWTH IN EMISSIONS
    growthYear1 = growthAccessor(selectedData.find((d) => d.year == year1));
    year1InnerDim.set(Math.round(Math.sqrt(areaScale(growthYear1))));

    // console.log({ year1InnerDim });
  }

  // -----------------------------------------------------------------------------
  // INTERACTION
  // -----------------------------------------------------------------------------

  // let sliderYear;
  // $: console.log({sliderYear});
  // $: console.log({ year1 });
</script>

<div>Treemap</div>

<!-- to fix issue with incorrect initial slide value display while data is loading -->


<div>
  <svg width="400" height="400">
    {#each processedDataTree.leaves() as leaf, leafIndex}
      <rect
        x={leaf.x0}
        y={leaf.y0}
        width={leaf.x1 - leaf.x0}
        height={leaf.y1 - leaf.y0}
        fill="none"
        stroke="black"
      />
    {/each}
  </svg>
</div>

{#if data.length > 0}
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
{/if}

<svg width="600" height="400">
  <rect x="0" y="0" width={$year1OuterDim} height={$year1OuterDim} />
  <rect
    x="0"
    y="0"
    width={$year1InnerDim}
    height={$year1InnerDim}
    fill="white"
  />
</svg>

<div>
  <svg width={maxRectWidth} height={maxRectHeight}>
    {#each data.filter((d) => d.year % 10 === 0) as d}
      <rect
        x="0"
        y="0"
        width={calcSquareDim(totalAccessor(d))}
        height={calcSquareDim(totalAccessor(d))}
        fill="none"
        stroke="black"
      />
    {/each}
  </svg>
</div>
