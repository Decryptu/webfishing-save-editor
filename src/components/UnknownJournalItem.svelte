<script lang="ts">
  import type { JournalItem } from "../game/types";
  import { createEventDispatcher } from "svelte";

  export let id: string;
  export let item: JournalItem;

  const dispatch = createEventDispatcher<{ remove: string }>();

  const rarities = ["Normal", "Shining", "Glistening", "Opulent", "Radiant", "Alpha"];
</script>

<fieldset class="grid unknownJournalItem">
  <div class="unknownItemHeader">
    <span class="unknownIcon">???</span>
    <div class="unknownItemInfo">
      <span class="unknownName">Unknown Entry</span>
      <code class="unknownId">{id}</code>
    </div>
  </div>

  <div>
    <p class="rarityLabel">Rarities caught:</p>
    {#each item.quality.value as q}
      <span class="rarityBadge">{rarities[q.value] ?? `#${q.value}`}</span>
    {/each}
    {#if item.quality.value.length === 0}
      <span class="rarityBadge">None</span>
    {/if}
  </div>

  <div>
    <span>Record: {item.record.value}</span>
    <br />
    <span>Caught: {item.count.value}</span>
  </div>

  <div>
    <button class="removeBtn" on:click={() => dispatch("remove", id)}>Remove</button>
  </div>
</fieldset>

<style>
  .unknownJournalItem {
    align-items: center;
    border: 2px dashed var(--color-danger);
    background: rgba(173, 10, 35, 0.15);
  }

  .unknownItemHeader {
    display: flex;
    align-items: center;
    margin-bottom: var(--spacing-sm);
    gap: var(--spacing-sm);
  }

  .unknownIcon {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 24px;
    height: 24px;
    font-weight: bold;
    font-size: var(--font-size-xs);
    background: var(--color-danger);
    color: var(--color-sky);
  }

  .unknownItemInfo {
    display: flex;
    flex-direction: column;
  }

  .unknownName {
    font-weight: bold;
    color: var(--color-danger);
    font-size: var(--font-size-xs);
  }

  .unknownId {
    font-size: var(--font-size-xs);
    opacity: 0.8;
  }

  .rarityLabel {
    margin-bottom: var(--spacing-xs);
    font-size: var(--font-size-xs);
  }

  .rarityBadge {
    display: inline-block;
    padding: var(--spacing-xs) var(--spacing-sm);
    margin: 2px;
    font-size: var(--font-size-xs);
    background: var(--color-border);
    border: 2px solid var(--color-border);
  }

  .removeBtn {
    background: var(--color-danger);
    margin: 0;
  }
</style>
