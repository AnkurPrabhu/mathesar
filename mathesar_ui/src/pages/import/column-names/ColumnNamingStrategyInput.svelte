<script lang="ts">
  import type { ComponentProps } from 'svelte';

  import { RadioGroup } from '@mathesar-component-library';
  import LabelYesHeaderRow from './LabelYesHeaderRow.svelte';
  import LabelNoHeaderRow from './LabelNoHeaderRow.svelte';

  type $$Props = ComponentProps<RadioGroup<boolean>>;

  /** True when the first row is used as the column names */
  export let value = true;
</script>

<RadioGroup
  bind:value
  on:change
  boxed
  options={[true, false]}
  label="Column Names"
  getRadioLabel={(v) =>
    v ? { component: LabelYesHeaderRow } : { component: LabelNoHeaderRow }}
  {...$$restProps}
/>
