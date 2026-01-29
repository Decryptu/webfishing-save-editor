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
    border: 2px dashed var(--pico-del-color, #e53935);
    background: color-mix(in srgb, var(--pico-del-color, #e53935) 8%, transparent);
  }

  .unknownItemHeader {
    display: flex;
    align-items: center;
    margin-bottom: var(--pico-spacing);
    gap: var(--pico-spacing);
  }

  .unknownIcon {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 1.5rem;
    height: 1.5rem;
    font-weight: bold;
    font-size: 0.7rem;
    background: var(--pico-del-color, #e53935);
    color: white;
    border-radius: 4px;
  }

  .unknownItemInfo {
    display: flex;
    flex-direction: column;
  }

  .unknownName {
    font-weight: bold;
    color: var(--pico-del-color, #e53935);
  }

  .unknownId {
    font-size: 0.8em;
    opacity: 0.8;
  }

  .rarityLabel {
    margin-bottom: 0.25rem;
    font-size: 0.85em;
  }

  .rarityBadge {
    display: inline-block;
    padding: 0.1rem 0.4rem;
    margin: 0.1rem;
    font-size: 0.8em;
    border-radius: 4px;
    background: var(--pico-muted-border-color);
  }

  .removeBtn {
    background: var(--pico-del-color, #e53935);
    border-color: var(--pico-del-color, #e53935);
    margin: 0;
  }
</style>
