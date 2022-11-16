<script>
  //import { tick } from "svelte";
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";
  import { createVirtualizer, createWindowVirtualizer } from "@tanstack/svelte-virtual";

  import records0 from "./tgif";
  $: records = records0.map(([url, description], index) => [index + 1, url, description]);

  let inputText = "";
  let filterText = "";
  $: filteredRecords = filterText ? records.filter(([_index, _url, desc]) => desc.includes(filterText)) : records;
  $: filteredCount = filteredRecords.length;

  const countTween = tweened(0, {
    duration: 400,
    easing: cubicOut,
  });
  $: countTween.set(filteredCount);

  //let scrolling = false;

  /** @type HTMLElement */
  let containerElement;

  const ROW_SIZE = 120;

  $: rowVirtualizer = createVirtualizer({
    count: filteredCount,
    getScrollElement: () => containerElement,
    estimateSize: () => ROW_SIZE,
    //onChange: () => {
    //  containerElement.classList.add("change");
    //  tick().then(() => {
    //    //setTimeout(() => {
    //    requestAnimationFrame(() => {
    //      containerElement.classList.remove("change");
    //    });
    //    //}, 1000);
    //  });
    //},
  });

  /** @param {Number} num */
  function formatNumber(num) {
    return new Intl.NumberFormat().format(num);
  }

  /** @param {string} desc */
  function highlight(desc) {
    if (!filterText) return desc;
    const splits = desc.split(filterText);
    return `${splits.join(`<em>${filterText}</em>`)}`;
  }
</script>

<main>
  <h1>TGIF Dataset</h1>

  <form action="" on:submit|preventDefault={() => (filterText = inputText.toLowerCase())}>
    <input type="text" placeholder="Filter..." bind:value={inputText} />
    <button>Filter</button>
  </form>

  <p>
    {formatNumber(Math.floor($countTween))} / {formatNumber(records.length)} data
  </p>

  <div class="table-container virtual" bind:this={containerElement}>
    <!--class:change={scrolling}-->
    <!--on:scroll={() => { scrolling = true; }}-->
    <table style:height={$rowVirtualizer.getTotalSize() + "px"} style:--row-height={ROW_SIZE}>
      {#each $rowVirtualizer.getVirtualItems() as vRow}
        {@const [index, url, description] = filteredRecords[vRow.index]}
        <tr style:transform={`translateY(${vRow.start}px)`}>
          <td class="index">#{index}</td>
          <td><img loading="lazy" src={url} alt={description} /></td>
          <td
            ><p>
              {@html highlight(description)}
            </p></td
          >
        </tr>
      {/each}
    </table>
  </div>
</main>

<style>
  main {
    height: 100vh;
    display: flex;
    padding: 8px;
    flex-direction: column;
    box-sizing: border-box;
  }
  h1 {
    margin: 8px 0;
    font-size: 1.5rem;
  }

  .table-container {
    flex: auto;
    overflow: auto;
  }
  .table-container.virtual > table {
    position: relative;
    width: 100%;
  }
  .table-container.virtual > table > tr {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
  }
  /*.table-container:global(.change) img {
    opacity: 0.3;
  }*/

  tr {
    height: var(--row-height) px;
  }
  tr:nth-of-type(odd) {
    background-color: #f5f5f5;
  }
  td {
    padding: 0;
  }
  td.index {
    width: 80px;
    padding-right: 8px;
    text-align: right;
    box-sizing: border-box;
    font-size: 16px;
  }

  img {
    width: 130px;
    height: 100px;
  }

  p {
    margin: 0;
    padding: 8px;
    box-sizing: border-box;
    font-size: 16px;
  }
  p :global(em) {
    font-style: normal;
    font-weight: bold;
  }
</style>
