<script lang="ts">
  import type { WebfishingSave } from "../game/types";
  import things from "../game/things";
  import { GodotVariantType, type GodotString } from "../lib/types";
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

  type CorruptionDetails = {
    journalEntries: { section: string; ids: string[] }[];
    equippedCosmetics: string[];
    unlockedCosmetics: string[];
  };

  function checkCorrupt(): CorruptionDetails | null {
    const details: CorruptionDetails = {
      journalEntries: [],
      equippedCosmetics: [],
      unlockedCosmetics: []
    };

    for (const [section, value] of Object.entries(dictWithoutSpecial(save.value.journal.value))) {
      const unknownIds = Object.keys(value.value)
        .filter((x) => !x.startsWith("__saveEditor") && !(x in things));
      if (unknownIds.length > 0) {
        details.journalEntries.push({ section, ids: unknownIds });
      }
    }

    const cosmeticsEquipped = Object.values(dictWithoutSpecial(save.value.cosmetics_equipped.value))
      .map((x) => (x.$type === GodotVariantType.String ? [x] : x.value) as GodotString[])
      .flat()
      .map((x) => x.value);
    details.equippedCosmetics = cosmeticsEquipped.filter((x) => !(x in things));

    const cosmeticsUnlocked = save.value.cosmetics_unlocked.value.map((x) => x.value).flat();
    details.unlockedCosmetics = cosmeticsUnlocked.filter((x) => !(x in things));

    const hasCorruption =
      details.journalEntries.length > 0 ||
      details.equippedCosmetics.length > 0 ||
      details.unlockedCosmetics.length > 0;

    if (hasCorruption) {
      console.log("Corruption details:", details);
      return details;
    }

    return null;
  }

  let corruption = checkCorrupt();

  function fixCorrupt() {
    for (const [key, value] of Object.entries(save.value.journal.value)) {
      if (key.startsWith("__saveEditor")) continue;

      for (const [key2, value2] of Object.entries(value.value)) {
        if (key2.startsWith("__saveEditor")) continue;
        if (!(key2 in things)) {
          console.log(`Deleting ${key2} from journal`);
          delete value.value[key2];
        }
      }
    }

    const untypedCosmeticsEquipped: Record<any, any> = save.value.cosmetics_equipped.value;
    for (const [key, value] of Object.entries(save.value.cosmetics_equipped.value)) {
      if (key.startsWith("__saveEditor")) continue;

      if (value.$type === GodotVariantType.String) {
        if (!(value.value in things)) {
          console.log(`Deleting ${value.value} from equipped cosmetics`);
          const fallback = fallbackCosm[key];
          untypedCosmeticsEquipped[key] = Array.isArray(fallback) ? array(fallback.map(string)) : string(fallback);
        }
      } else {
        const cloned = [...value.value];

        for (let i = 0; i < cloned.length; i++) {
          const item = cloned[i];
          if (!(item.value in things)) {
            console.log(`Deleting ${item.value} from equipped cosmetics`);
            value.value.splice(value.value.indexOf(item), 1);
          }
        }
      }
    }

    const cloned = [...save.value.cosmetics_unlocked.value];
    for (let i = 0; i < cloned.length; i++) {
      const item = cloned[i];
      if (!(item.value in things)) {
        console.log(`Deleting ${item.value} from unlocked cosmetics`);
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

    {#if corruption.journalEntries.length > 0}
      <details open>
        <summary>Unknown journal entries</summary>
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

    {#if corruption.equippedCosmetics.length > 0}
      <details>
        <summary>Unknown equipped cosmetics</summary>
        <ul>
          {#each corruption.equippedCosmetics as id}
            <li><code>{id}</code></li>
          {/each}
        </ul>
      </details>
    {/if}

    {#if corruption.unlockedCosmetics.length > 0}
      <details>
        <summary>Unknown unlocked cosmetics</summary>
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
