<script lang="ts">
  import type { WebfishingSave } from "../game/types";
  import Section from "../components/Section.svelte";
  import { lures, baits } from "../game/lure";
  import { string } from "../lib/godot";
  import { iconsDir } from "../lib/site";

  export let save: WebfishingSave;

  function setLure(id: string, value: boolean) {
    if (save.value.lure_unlocked.value.find((i) => i.value === id)) {
      save.value.lure_unlocked.value = save.value.lure_unlocked.value.filter((i) => i.value !== id);
    } else {
      save.value.lure_unlocked.value.push(string(id));
    }
  }

  function setLureWrapped(id: string, event: Event) {
    setLure(id, (event.target as HTMLInputElement).checked);
  }

  function setBait(id: string, value: boolean) {
    if (save.value.bait_unlocked.value.find((i) => i.value === id)) {
      save.value.bait_unlocked.value = save.value.bait_unlocked.value.filter((i) => i.value !== id);
    } else {
      save.value.bait_unlocked.value.push(string(id));
    }
  }

  function setBaitWrapped(id: string, event: Event) {
    setBait(id, (event.target as HTMLInputElement).checked);
  }
</script>

<Section title="Baits & lures">
  <div class="luresGrid">
    {#each Object.keys(baits) as bait, i}
      <div class="lureItem">
        <input
          type="checkbox"
          id={`bait-${bait}`}
          checked={save.value.bait_unlocked.value.find((l) => l.value === bait) != null}
          on:change={(e) => setBaitWrapped(bait, e)}
        />
        <img src={`${iconsDir}/${baits[bait].icon}`} alt={baits[bait].name} class="icon" />
        <label for={`bait-${bait}`}>{baits[bait].name}</label>
      </div>
    {/each}

    {#each Object.keys(lures) as lure, i}
      <div class="lureItem">
        <input
          type="checkbox"
          id={`lure-${lure}`}
          checked={save.value.lure_unlocked.value.find((l) => l.value === lure) != null}
          on:change={(e) => setLureWrapped(lure, e)}
        />
        <img src={`${iconsDir}/${lures[lure].icon}`} alt={lures[lure].name} class="icon" />
        <label for={`lure-${lure}`}>{lures[lure].name}</label>
      </div>
    {/each}
  </div>
</Section>

<style>
  .luresGrid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: var(--spacing-sm);
  }

  .lureItem {
    display: flex;
    align-items: center;
    gap: var(--spacing-sm);
    padding: var(--spacing-xs);
    background-color: rgba(42, 42, 42, 0.5);
    border: 2px solid var(--color-border);
  }

  .lureItem:hover {
    background-color: rgba(49, 141, 112, 0.1);
    border-color: var(--color-primary);
  }

  .icon {
    max-width: 32px;
    max-height: 32px;
    border: 2px solid var(--color-border);
    background-color: var(--color-bg-light);
    padding: 2px;
  }

  .lureItem label {
    flex: 1;
    margin: 0;
    font-size: var(--font-size-xs);
    cursor: pointer;
  }

  @media (max-width: 768px) {
    .luresGrid {
      grid-template-columns: 1fr;
    }
  }
</style>
