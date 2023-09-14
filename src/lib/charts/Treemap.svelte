<script>
  export let data;

  import { tidy, mutate, filter, lag, mutateWithSummary } from "@tidyjs/tidy";

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
      mutateWithSummary({
        total_ghg_emissions_years_ago_1: lag("total_ghg_emissions", {
          n: years_ago_1,
        }),
        total_ghg_emissions_years_ago_2: lag("total_ghg_emissions", {
          n: years_ago_2,
        }),
      }),
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
</script>

<div>Treemap</div>
