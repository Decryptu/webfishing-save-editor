<script lang="ts">
  import { items } from "../game/things";
  import type { WebfishingSave } from "../game/types";
  import { GodotVariantType } from "../lib/types";
  import JournalItem from "./JournalItem.svelte";
  import UnknownJournalItem from "./UnknownJournalItem.svelte";

  export let save: WebfishingSave;
  export let sectionName: string;

  let typedSectionName = sectionName as keyof WebfishingSave["value"]["journal"]["value"];
  let section = save.value.journal.value[typedSectionName];

  // Get all non-internal keys
  const allEntryKeys = Object.keys(section.value).filter((key) => !key.startsWith("__saveEditor"));

  // Known fish entries: key is in items AND value is a proper dictionary with JournalItem fields
  const knownKeys = allEntryKeys.filter((key) => {
    if (items[key] == null) return false;
    const entry = (section.value as Record<string, any>)[key];
    return entry && typeof entry === "object" && entry.$type === GodotVariantType.Dictionary;
  });

  // Everything else is unknown/corrupted
  let unknownKeys = allEntryKeys.filter((key) => !knownKeys.includes(key));

  function removeEntry(event: CustomEvent<string>) {
    const id = event.detail;
    delete section.value[id];
    // Also remove from __saveEditor_order if present
    const order = (section.value as any).__saveEditor_order;
    if (Array.isArray(order)) {
      const idx = order.indexOf(id);
      if (idx !== -1) order.splice(idx, 1);
    }
    unknownKeys = unknownKeys.filter((k) => k !== id);
  }

  const sectionValueAny: Record<string, any> = section.value;

  // Check if an unknown entry has a valid JournalItem shape (for display purposes)
  function hasJournalItemShape(key: string): boolean {
    try {
      const entry = sectionValueAny[key];
      if (!entry || typeof entry !== "object") return false;
      if (entry.$type !== GodotVariantType.Dictionary) return false;
      const val = entry.value;
      return val && "count" in val && "quality" in val && "record" in val;
    } catch {
      return false;
    }
  }

  function getRawEntryJson(key: string): string {
    try {
      return JSON.stringify(sectionValueAny[key], null, 2).slice(0, 200);
    } catch {
      return "(unable to serialize)";
    }
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
      {#if hasJournalItemShape(key)}
        <UnknownJournalItem id={key} item={section.value[key].value} on:remove={removeEntry} />
      {:else}
        <fieldset class="malformedEntry">
          <div>
            <span class="unknownIcon">???</span>
            <strong>Corrupted Entry</strong>
            <code>{key}</code>
            <br />
            <small>Raw: <code>{getRawEntryJson(key)}</code></small>
          </div>
          <button class="removeBtn" on:click={() => removeEntry(new CustomEvent("remove", { detail: key }))}>Remove</button>
        </fieldset>
      {/if}
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

  .malformedEntry {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    border: 2px dashed var(--pico-del-color, #e53935);
    background: color-mix(in srgb, var(--pico-del-color, #e53935) 8%, transparent);
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
    margin-right: 0.5rem;
  }

  .removeBtn {
    background: var(--pico-del-color, #e53935);
    border-color: var(--pico-del-color, #e53935);
    margin: 0;
    white-space: nowrap;
  }
</style>
