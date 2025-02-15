<script lang="ts">
  import { Help, isDefinedNonNullable } from '@mathesar-component-library';
  import type {
    JoinableTable,
    JoinableTablesResult,
  } from '@mathesar/api/types/tables/joinable_tables';
  import NameWithIcon from '@mathesar/components/NameWithIcon.svelte';
  import { iconRecord } from '@mathesar/icons';
  import { tables } from '@mathesar/stores/tables';
  import TableWidget from './TableWidget.svelte';

  export let recordPk: string;
  export let recordSummary: string;
  export let joinableTablesResult: JoinableTablesResult;

  $: columnNameMap = new Map(
    Object.entries(joinableTablesResult.columns).map(([columnId, column]) => [
      parseInt(columnId, 10),
      column.name,
    ]),
  );

  function buildWidgetInput(joinableTable: JoinableTable) {
    const table = $tables.data.get(joinableTable.target);
    if (!table) return undefined;
    const id = joinableTable.jp_path[0].slice(-1)[0];
    const name = columnNameMap.get(id) ?? '(unknown column)';
    return { table, fkColumn: { id, name } };
  }

  $: tableWidgetInputs = joinableTablesResult.joinable_tables
    .filter((joinableTable) => joinableTable.multiple_results)
    .map(buildWidgetInput)
    .filter(isDefinedNonNullable)
    .sort((a, b) => a.table.name.localeCompare(b.table.name));
</script>

{#if tableWidgetInputs.length}
  <div class="widgets-area">
    <h2 class="passthrough">
      Related Records
      <Help>
        Each of the following records links to
        <NameWithIcon icon={iconRecord} truncate={false}>
          <strong>{recordSummary}</strong>
        </NameWithIcon>
        from another table.
      </Help>
    </h2>
    <div class="widgets">
      {#each tableWidgetInputs as { table, fkColumn } (`${table.id}-${fkColumn.id}`)}
        <section class="table-widget-positioner">
          <TableWidget {recordPk} {table} {fkColumn} />
        </section>
      {/each}
    </div>
  </div>
{:else}
  <div class="no-widgets" />
{/if}

<style lang="scss">
  .widgets-area {
    background: var(--slate-50);
    border-top: 1px solid var(--slate-300);
  }
  .no-widgets {
    background: var(--sand-200);
  }
  h2 {
    padding: var(--size-small);
    border-bottom: 1px solid var(--slate-200);
    font-weight: 600;
    font-size: var(--size-large);
    background: var(--white);
  }
  .widgets {
    padding: var(--size-small);

    :global(.sheet) {
      background: var(--white);
    }

    :global(.sheet [data-sheet-element='header']) {
      background: var(--slate-100);
    }
  }

  .table-widget-positioner {
    margin: 4rem 0;
    &:first-child {
      margin-top: 0;
    }
    &:last-child {
      margin-bottom: 0;
    }
  }
</style>
