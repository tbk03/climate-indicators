<script>
  export let sliderValue;
</script>

<div class="range-container sticky">
  <div class="range-slider">
    <input type="range" min="0" max="100" bind:value={sliderValue} />
  </div>
</div>

<style>
  .range-container {
    background: white;
  }

  .range-slider {
    display: flex;
    justify-content: center;
  }

  div.sticky {
    position: -webkit-sticky; /* Safari */
    position: sticky;
    top: 0;
    z-index: 1;
  }
</style>
