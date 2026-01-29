<script lang="ts">
  import type { WebfishingSave } from "../game/types";
  import things from "../game/things";
  import { GodotVariantType, type GodotString, type GodotVariant } from "../lib/types";
  import { dictWithoutSpecial } from "../game/util";
  import { array, string } from "../lib/godot";

  export let save: WebfishingSave;

  const fallbackCosm: Record<string, string | string[]> = {
    species: "species_cat",
    pattern: "pattern_none",
    primary_color: "pcolor_white",
    secondary_color: "scolor_tan",
    hat: "hat_none",
    undershirt: "shirt_none",
    overshirt: "overshirt_none",
    title: "title_rank_1",
    bobber: "bobber_default",
    eye: "eye_halfclosed",
    nose: "nose_cat",
    mouth: "mouth_default",
    accessory: [],
    tail: "tail_cat",
    legs: "legs_none"
  };

  const sectionNames: Record<string, string> = {
    lake: "Lake",
    ocean: "Ocean",
    rain: "Rain",
    water_trash: "Trash",
    alien: "Alien",
    void: "Void"
  };

  // All known top-level save fields from the WebfishingSave type + newer game fields
  const knownSaveFields = new Set([
    "bait_inv", "bait_selected", "bait_unlocked", "buddy_level", "buddy_speed", "cash_total",
    "completed_quests", "cosmetics_equipped", "cosmetics_unlocked", "fish_caught", "guitar_shapes",
    "hidden_players", "hotbar", "inbound_mail", "inventory", "journal",
    "level", "loan_left", "loan_level", "locked_refs", "lure_selected", "lure_unlocked",
    "max_bait", "money", "muted_players", "new_cosmetics", "player_options",
    "quests", "recorded_time", "rod_chance", "rod_luck", "rod_power", "rod_speed",
    "saved_aqua_fish", "saved_tags", "shop", "version", "voice_pitch", "voice_speed", "xp"
  ]);

  const variantTypeName: Record<number, string> = {
    [GodotVariantType.Nil]: "Nil",
    [GodotVariantType.Bool]: "Bool",
    [GodotVariantType.Int]: "Int",
    [GodotVariantType.Real]: "Real",
    [GodotVariantType.String]: "String",
    [GodotVariantType.Vector2]: "Vector2",
    [GodotVariantType.Dictionary]: "Dictionary",
    [GodotVariantType.Array]: "Array"
  };

  type UnknownSaveField = {
    key: string;
    typeName: string;
    preview: string;
  };

  type CorruptionDetails = {
    journalEntries: { section: string; ids: string[] }[];
    malformedSections: { key: string; reason: string }[];
    equippedCosmetics: string[];
    unlockedCosmetics: string[];
    unknownSaveFields: UnknownSaveField[];
  };

  function getVariantPreview(val: any): string {
    if (!val || typeof val !== "object") return String(val);
    if (val.$type === GodotVariantType.String) return `"${val.value}"`;
    if (val.$type === GodotVariantType.Int || val.$type === GodotVariantType.Real) return String(val.value);
    if (val.$type === GodotVariantType.Bool) return String(val.value);
    if (val.$type === GodotVariantType.Nil) return "null";
    if (val.$type === GodotVariantType.Array) return `Array(${val.value.length})`;
    if (val.$type === GodotVariantType.Dictionary) {
      const keys = Object.keys(val.value).filter((k: string) => !k.startsWith("__saveEditor"));
      return `Dict{${keys.slice(0, 5).join(", ")}${keys.length > 5 ? ", ..." : ""}}`;
    }
    return JSON.stringify(val).slice(0, 100);
  }

  function checkCorrupt(): CorruptionDetails | null {
    const details: CorruptionDetails = {
      journalEntries: [],
      malformedSections: [],
      equippedCosmetics: [],
      unlockedCosmetics: [],
      unknownSaveFields: []
    };

    // === Scan ALL top-level save keys for unknown fields ===
    const saveValue = save.value as Record<string, any>;
    const allSaveKeys = Object.keys(saveValue).filter((x) => !x.startsWith("__saveEditor"));
    console.log("[Save Editor] All top-level save keys:", allSaveKeys);

    const unknownTopKeys = allSaveKeys.filter((k) => !knownSaveFields.has(k));
    if (unknownTopKeys.length > 0) {
      console.log("[Save Editor] UNKNOWN top-level save keys:", unknownTopKeys);
      for (const key of unknownTopKeys) {
        const val = saveValue[key];
        const typeName = val && typeof val === "object" && val.$type !== undefined
          ? (variantTypeName[val.$type] ?? `Type#${val.$type}`)
          : typeof val;
        details.unknownSaveFields.push({
          key,
          typeName,
          preview: getVariantPreview(val)
        });
        console.log(`[Save Editor] Unknown save field "${key}":`, val);
      }
    }

    // === Scan journal sections ===
    const journalValue = save.value.journal.value;
    const allJournalKeys = Object.keys(journalValue).filter((x) => !x.startsWith("__saveEditor"));
    console.log("[Save Editor] Journal sections found:", allJournalKeys);

    for (const sectionKey of allJournalKeys) {
      const section = (journalValue as Record<string, any>)[sectionKey];

      if (!section || typeof section !== "object" || section.$type !== GodotVariantType.Dictionary) {
        console.log(`[Save Editor] Malformed journal section "${sectionKey}":`, section);
        details.malformedSections.push({
          key: sectionKey,
          reason: !section
            ? "null/undefined value"
            : section.$type !== undefined
              ? `unexpected type ${variantTypeName[section.$type] ?? section.$type}`
              : "not a dictionary"
        });
        continue;
      }

      const sectionValue = section.value;
      if (!sectionValue || typeof sectionValue !== "object") {
        console.log(`[Save Editor] Section "${sectionKey}" has no value:`, section);
        details.malformedSections.push({ key: sectionKey, reason: "section has no inner value" });
        continue;
      }

      const entryKeys = Object.keys(sectionValue).filter((x: string) => !x.startsWith("__saveEditor"));
      console.log(`[Save Editor] Section "${sectionKey}" entries:`, entryKeys);

      const unknownIds: string[] = [];
      for (const entryKey of entryKeys) {
        if (!(entryKey in things)) {
          unknownIds.push(entryKey);
        }
        const entry = sectionValue[entryKey];
        if (entry && typeof entry === "object" && entry.$type !== GodotVariantType.Dictionary) {
          console.log(
            `[Save Editor] Entry "${entryKey}" in section "${sectionKey}" is not a dictionary:`,
            entry
          );
          if (!unknownIds.includes(entryKey)) {
            unknownIds.push(entryKey);
          }
        }
      }

      if (unknownIds.length > 0) {
        details.journalEntries.push({ section: sectionKey, ids: unknownIds });
      }
    }

    // === Check cosmetics ===
    try {
      const cosmeticsEquipped = Object.values(dictWithoutSpecial(save.value.cosmetics_equipped.value))
        .map((x: any) => (x.$type === GodotVariantType.String ? [x] : x.value) as GodotString[])
        .flat()
        .map((x) => x.value);
      details.equippedCosmetics = cosmeticsEquipped.filter((x) => !(x in things));
    } catch (e) {
      console.log("[Save Editor] Error checking equipped cosmetics:", e);
    }

    try {
      const cosmeticsUnlocked = save.value.cosmetics_unlocked.value.map((x) => x.value).flat();
      details.unlockedCosmetics = cosmeticsUnlocked.filter((x) => !(x in things));
    } catch (e) {
      console.log("[Save Editor] Error checking unlocked cosmetics:", e);
    }

    const hasCorruption =
      details.journalEntries.length > 0 ||
      details.malformedSections.length > 0 ||
      details.equippedCosmetics.length > 0 ||
      details.unlockedCosmetics.length > 0 ||
      details.unknownSaveFields.length > 0;

    if (hasCorruption) {
      console.log("[Save Editor] Corruption details:", details);
      return details;
    }

    console.log("[Save Editor] No corruption detected.");
    return null;
  }

  let corruption = checkCorrupt();

  function removeUnknownSaveField(key: string) {
    const saveValue = save.value as Record<string, any>;
    delete saveValue[key];
    const order = (saveValue as any).__saveEditor_order;
    if (Array.isArray(order)) {
      const idx = order.indexOf(key);
      if (idx !== -1) order.splice(idx, 1);
    }
    if (corruption) {
      corruption.unknownSaveFields = corruption.unknownSaveFields.filter((f) => f.key !== key);
      corruption = corruption; // trigger reactivity
    }
  }

  function fixCorrupt() {
    // Remove unknown top-level save fields
    for (const { key } of corruption?.unknownSaveFields ?? []) {
      console.log(`Deleting unknown save field "${key}"`);
      removeUnknownSaveField(key);
    }

    // Remove malformed sections from journal
    for (const { key } of corruption?.malformedSections ?? []) {
      console.log(`Deleting malformed journal section "${key}"`);
      delete (save.value.journal.value as Record<string, any>)[key];
      const order = (save.value.journal.value as any).__saveEditor_order;
      if (Array.isArray(order)) {
        const idx = order.indexOf(key);
        if (idx !== -1) order.splice(idx, 1);
      }
    }

    // Remove unknown entries within journal sections
    for (const [key, value] of Object.entries(save.value.journal.value)) {
      if (key.startsWith("__saveEditor")) continue;

      if (!value || typeof value !== "object" || (value as any).$type !== GodotVariantType.Dictionary) {
        continue;
      }

      for (const [key2] of Object.entries(value.value)) {
        if (key2.startsWith("__saveEditor")) continue;
        if (!(key2 in things)) {
          console.log(`Deleting "${key2}" from journal section "${key}"`);
          delete value.value[key2];
        }
      }
    }

    // Fix equipped cosmetics
    const untypedCosmeticsEquipped: Record<any, any> = save.value.cosmetics_equipped.value;
    for (const [key, value] of Object.entries(save.value.cosmetics_equipped.value)) {
      if (key.startsWith("__saveEditor")) continue;

      if (value.$type === GodotVariantType.String) {
        if (!(value.value in things)) {
          console.log(`Replacing "${value.value}" in equipped cosmetics with fallback`);
          const fallback = fallbackCosm[key];
          untypedCosmeticsEquipped[key] = Array.isArray(fallback) ? array(fallback.map(string)) : string(fallback);
        }
      } else {
        const cloned = [...value.value];
        for (let i = 0; i < cloned.length; i++) {
          const item = cloned[i];
          if (!(item.value in things)) {
            console.log(`Deleting "${item.value}" from equipped cosmetics`);
            value.value.splice(value.value.indexOf(item), 1);
          }
        }
      }
    }

    // Fix unlocked cosmetics
    const cloned = [...save.value.cosmetics_unlocked.value];
    for (let i = 0; i < cloned.length; i++) {
      const item = cloned[i];
      if (!(item.value in things)) {
        console.log(`Deleting "${item.value}" from unlocked cosmetics`);
        save.value.cosmetics_unlocked.value.splice(save.value.cosmetics_unlocked.value.indexOf(item), 1);
      }
    }

    corruption = null;
  }
</script>

{#if corruption}
  <article>
    <header>Potentially corrupt save</header>
    <p>
      Your save contains data that doesn't match any known game content. This is often caused by a game bug (like a
      phantom "???" fish entry in the journal), a broken mod, or new game content not yet supported by this editor.
    </p>

    {#if corruption.unknownSaveFields.length > 0}
      <details open>
        <summary>Unknown save fields ({corruption.unknownSaveFields.length})</summary>
        <p>These fields exist in your save but are not recognized by the editor. They may be causing the phantom "???" fish.</p>
        <ul>
          {#each corruption.unknownSaveFields as field}
            <li>
              <code>{field.key}</code> ({field.typeName}) &mdash; {field.preview}
              <button class="inlineRemoveBtn" on:click={() => removeUnknownSaveField(field.key)}>Remove</button>
            </li>
          {/each}
        </ul>
      </details>
    {/if}

    {#if corruption.journalEntries.length > 0}
      <details open>
        <summary>Unknown journal entries ({corruption.journalEntries.reduce((a, e) => a + e.ids.length, 0)})</summary>
        <ul>
          {#each corruption.journalEntries as entry}
            <li>
              <strong>{sectionNames[entry.section] ?? entry.section}</strong>:
              {#each entry.ids as id, i}
                <code>{id}</code>{i < entry.ids.length - 1 ? ", " : ""}
              {/each}
            </li>
          {/each}
        </ul>
        <p>
          <small>You can also scroll down to the Journal section to see and individually remove these entries.</small>
        </p>
      </details>
    {/if}

    {#if corruption.malformedSections.length > 0}
      <details open>
        <summary>Malformed journal sections ({corruption.malformedSections.length})</summary>
        <ul>
          {#each corruption.malformedSections as section}
            <li>
              <code>{section.key}</code> &mdash; {section.reason}
            </li>
          {/each}
        </ul>
      </details>
    {/if}

    {#if corruption.equippedCosmetics.length > 0}
      <details>
        <summary>Unknown equipped cosmetics ({corruption.equippedCosmetics.length})</summary>
        <ul>
          {#each corruption.equippedCosmetics as id}
            <li><code>{id}</code></li>
          {/each}
        </ul>
      </details>
    {/if}

    {#if corruption.unlockedCosmetics.length > 0}
      <details>
        <summary>Unknown unlocked cosmetics ({corruption.unlockedCosmetics.length})</summary>
        <ul>
          {#each corruption.unlockedCosmetics as id}
            <li><code>{id}</code></li>
          {/each}
        </ul>
      </details>
    {/if}

    <footer>
      <button on:click={fixCorrupt}>Fix all</button>
    </footer>
  </article>
{/if}

<style>
  .inlineRemoveBtn {
    display: inline;
    padding: 0.1rem 0.5rem;
    margin-left: 0.5rem;
    font-size: 0.8em;
    background: var(--pico-del-color, #e53935);
    border-color: var(--pico-del-color, #e53935);
    color: white;
  }
</style>
