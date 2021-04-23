<script>
  import { createEventDispatcher, onDestroy, onMount } from 'svelte';
  import { Grid } from 'ag-grid-community';
  import 'ag-grid-community/dist/styles/ag-grid.css';
  import 'ag-grid-community/dist/styles/ag-theme-alpine.css';

  const dispatch = createEventDispatcher();

  export let columnDefs;
  export let data;
  export let theme = 'alpine';
  export let options = {
    defaultColDef: {
      flex: 1,
      minWidth: 150,
      filter: true,
    },
    rowSelection: 'multiple',
  };
  export let loading = false;

  let themeUrl = `https://unpkg.com/ag-grid-community/dist/styles/ag-theme-${theme}.css`;
  let events = {};
  let ref;
  let grid;
  let api;

  const onSelectionChanged = (event) => {
    const selectedRows = api.getSelectedRows();
    dispatch('select', selectedRows);
    if (events.onSelectionChanged) events.onSelectionChanged(event);
  };

  const onCellValueChanged = (event) => {
    data[event.rowIndex] = event.data;
    dispatch('update', { row: event.rowIndex, data: event.data });

    if (events.onCellValueChanged) {
      events.onCellValueChanged(event);
    }
  };

  const onGridReady = (event) => {
    api = event.api;

    if (loading) {
      api.showLoadingOverlay();
    }

    if (events.onGridReady) {
      events.onGridReady(event);
    }
  };

  const updateData = (data) => {
    if (grid && api) {
      api.setRowData(data);
      api.setColumnDefs(columnDefs);
    }
  };

  onMount(() => {
    [
      'onSelectionChanged',
      'onCellValueChanged',
      'onGridReady',
    ].forEach((event) => {
      if (options[event]) {
        events[event] = options[event];
      }
    });

    grid = new Grid(ref, {
      ...options,
      columnDefs,
      rowData: data,
      onGridReady,
      onCellValueChanged,
      onSelectionChanged,
    });
  });

  onDestroy(() => {
    if (grid) {
      grid.destroy();
    }
  });

  $: updateData(data);
</script>

<style>
  .container {
    width: 100%;
    height: var(--grid-height, 800px);
  }
</style>

<svelte:head>
  {#if theme !== "alpine" && !Object.values(document.styleSheets).some((styleSheet) => styleSheet.href === themeUrl)}
    <link rel="stylesheet" href={themeUrl} />
  {/if}
</svelte:head>
<div class="container">
  <div bind:this={ref} style="height: 100%; width:100%" class="ag-theme-{theme}" />
</div>
