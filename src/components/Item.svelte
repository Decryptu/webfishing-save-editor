<script lang="ts">
  import { createEventDispatcher } from "svelte";
  import { items } from "../game/things";
  import type { InventoryItem } from "../game/types";
  import type { GodotCustomDictionary } from "../lib/types";
  import { iconsDir } from "../lib/site";

  const dispatch = createEventDispatcher();

  export let i: number;
  export let item: GodotCustomDictionary<InventoryItem>;
</script>

<div class="item">
  <div class="itemSelect">
    <img src={`${iconsDir}/${items[item.value.id.value]?.icon}`} alt={items[item.value.id.value]?.name} />
    <select bind:value={item.value.id.value}>
      {#each Object.keys(items) as id}
        <option value={id}>{items[id]?.name} ({id})</option>
      {/each}
    </select>

    <button on:click={() => dispatch("explode")}>Remove</button>
  </div>

  <fieldset class="grid">
    <div>
      <label for={`item-${i}-quality`}>Quality</label>
      <input type="number" bind:value={item.value.quality.value} id={`item-${i}-quality`} />
    </div>

    <div>
      <label for={`item-${i}-size`}>Size</label>
      <input type="number" bind:value={item.value.size.value} id={`item-${i}-size`} />
    </div>
  </fieldset>
</div>

<style>
  .item {
    display: flex;
    flex-direction: column;
    margin-bottom: var(--spacing-md);
  }

  .itemSelect {
    display: flex;
    align-items: center;
    margin-bottom: var(--spacing-md);
    gap: var(--spacing-md);
  }

  .itemSelect img {
    max-height: 48px;
    aspect-ratio: 1 / 1;
    border: 2px solid var(--color-border);
    background-color: var(--color-bg-light);
    padding: 2px;
  }

  .itemSelect select {
    margin-bottom: 0;
    flex: 1;
  }

  .itemSelect button {
    white-space: nowrap;
  }
</style>
