<script lang="ts">
  import type { WebfishingSave } from "../game/types";
  import { GodotVariantType } from "../lib/types";
  import { items } from "../game/things";
  import Section from "../components/Section.svelte";
  import JournalSection from "../components/JournalSection.svelte";

  export let save: WebfishingSave;

  const journalValue = save.value.journal.value as Record<string, any>;
  const allKeys = Object.keys(journalValue).filter((x) => !x.startsWith("__saveEditor"));

  // Separate valid sections (proper dictionaries) from malformed ones
  const sections = allKeys.filter((key) => {
    const val = journalValue[key];
    return val && typeof val === "object" && val.$type === GodotVariantType.Dictionary;
  });

  const malformedKeys = allKeys.filter((key) => {
    const val = journalValue[key];
    return !val || typeof val !== "object" || val.$type !== GodotVariantType.Dictionary;
  });

  const sectionNames: Record<string, string> = {
    lake: "Lake",
    ocean: "Ocean",
    rain: "Rain",
    water_trash: "Trash",
    alien: "Alien",
    void: "Void"
  };

  // Build debug info
  type SectionDebug = { key: string; name: string; entryKeys: string[]; unknownKeys: string[] };
  const debugSections: SectionDebug[] = sections.map((key) => {
    const sectionData = journalValue[key];
    const entryKeys = Object.keys(sectionData.value).filter((k: string) => !k.startsWith("__saveEditor"));
    const unknownKeys = entryKeys.filter((k: string) => !(k in items));
    return { key, name: sectionNames[key] ?? key, entryKeys, unknownKeys };
  });

  function removeMalformedSection(key: string) {
    delete journalValue[key];
    const order = (journalValue as any).__saveEditor_order;
    if (Array.isArray(order)) {
      const idx = order.indexOf(key);
      if (idx !== -1) order.splice(idx, 1);
    }
    malformedKeys.splice(malformedKeys.indexOf(key), 1);
    // Force Svelte reactivity
    save = save;
  }
</script>

<Section title="Journal">
  <div class="indented">
    {#each sections as key}
      <Section title={sectionNames[key] ?? key}>
        <JournalSection {save} sectionName={key} />
      </Section>
    {/each}

    {#if malformedKeys.length > 0}
      <Section title="Malformed Entries">
        <div class="malformedSection">
          <p>
            These journal entries are not valid sections and are likely corrupted.
            They may cause the "???" phantom fish in your game.
          </p>
          {#each malformedKeys as key}
            <fieldset class="malformedEntry">
              <div>
                <strong>Key:</strong> <code>{key}</code>
                <br />
                <strong>Raw value:</strong> <code>{JSON.stringify(journalValue[key], null, 2).slice(0, 200)}</code>
              </div>
              <button class="removeBtn" on:click={() => removeMalformedSection(key)}>Remove</button>
            </fieldset>
          {/each}
        </div>
      </Section>
    {/if}

    <Section title="Debug: Raw Journal Data">
      <div class="debugPanel">
        <p>
          If you don't see any corruption above but still have a "???" fish in-game,
          open your browser console (F12) and look for lines starting with <code>[Save Editor]</code>.
          This information can help diagnose the issue.
        </p>
        <table>
          <thead>
            <tr>
              <th>Section</th>
              <th>Entries</th>
              <th>Unknown</th>
              <th>All Keys</th>
            </tr>
          </thead>
          <tbody>
            {#each debugSections as section}
              <tr class:hasUnknown={section.unknownKeys.length > 0}>
                <td><strong>{section.name}</strong> <small>({section.key})</small></td>
                <td>{section.entryKeys.length}</td>
                <td>{section.unknownKeys.length}</td>
                <td><code class="keyList">{section.entryKeys.join(", ")}</code></td>
              </tr>
            {/each}
            {#each malformedKeys as key}
              <tr class="hasUnknown">
                <td><strong>{key}</strong> <small>(malformed)</small></td>
                <td>N/A</td>
                <td>N/A</td>
                <td><code>Not a valid section</code></td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    </Section>
  </div>
</Section>

<style>
  .indented {
    margin-left: 1rem;
  }

  .debugPanel table {
    font-size: 0.85em;
    width: 100%;
  }

  .debugPanel .keyList {
    font-size: 0.8em;
    word-break: break-all;
  }

  .hasUnknown {
    color: var(--pico-del-color, #e53935);
  }

  .malformedSection p {
    color: var(--pico-del-color, #e53935);
  }

  .malformedEntry {
    display: flex;
    align-items: center;
    justify-content: space-between;
    border: 2px dashed var(--pico-del-color, #e53935);
    background: color-mix(in srgb, var(--pico-del-color, #e53935) 8%, transparent);
  }

  .removeBtn {
    background: var(--pico-del-color, #e53935);
    border-color: var(--pico-del-color, #e53935);
    margin: 0;
    white-space: nowrap;
  }
</style>
