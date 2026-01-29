<script lang="ts">
  import { items } from "../game/things";
  import type { WebfishingSave } from "../game/types";
  import JournalItem from "./JournalItem.svelte";
  import UnknownJournalItem from "./UnknownJournalItem.svelte";

  export let save: WebfishingSave;
  export let sectionName: string;

  let typedSectionName = sectionName as keyof WebfishingSave["value"]["journal"]["value"];
  let section = save.value.journal.value[typedSectionName];

  const knownKeys = Object.keys(section.value).filter((key) => !key.startsWith("_") && items[key] != null);

  let unknownKeys = Object.keys(section.value).filter((key) => !key.startsWith("_") && items[key] == null);

  function removeEntry(event: CustomEvent<string>) {
    const id = event.detail;
    delete section.value[id];
    unknownKeys = unknownKeys.filter((k) => k !== id);
  }
</script>

{#each knownKeys as key}
  <JournalItem id={key} item={section.value[key].value} />
{/each}

{#if unknownKeys.length > 0}
  <div class="unknownSection">
    <p class="unknownHeader">
      {unknownKeys.length} unknown/corrupted {unknownKeys.length === 1 ? "entry" : "entries"} found in this section:
    </p>
    {#each unknownKeys as key (key)}
      <UnknownJournalItem id={key} item={section.value[key].value} on:remove={removeEntry} />
    {/each}
  </div>
{/if}

<style>
  .unknownSection {
    margin-top: 1rem;
  }

  .unknownHeader {
    font-weight: bold;
    color: var(--pico-del-color, #e53935);
    margin-bottom: 0.5rem;
  }
</style>
