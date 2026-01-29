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
    margin-top: var(--spacing-md);
  }

  .unknownHeader {
    font-weight: bold;
    color: var(--color-danger);
    margin-bottom: var(--spacing-sm);
  }

  .malformedEntry {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: var(--spacing-md);
    border: 2px dashed var(--color-danger);
    background: rgba(173, 10, 35, 0.15);
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
    margin-right: var(--spacing-sm);
  }

  .removeBtn {
    background: var(--color-danger);
    margin: 0;
    white-space: nowrap;
  }
</style>
