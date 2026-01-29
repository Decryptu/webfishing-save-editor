<script lang="ts">
  import type { WebfishingSave } from "../game/types";
  import Section from "../components/Section.svelte";
  import { cosmetics } from "../game/things";
  import { string } from "../lib/godot";
  import { iconsDir } from "../lib/site";

  export let save: WebfishingSave;

  function setCosmetic(id: string, value: boolean) {
    if (save.value.cosmetics_unlocked.value.find((i) => i.value === id)) {
      save.value.cosmetics_unlocked.value = save.value.cosmetics_unlocked.value.filter((i) => i.value !== id);
    } else {
      save.value.cosmetics_unlocked.value.push(string(id));
    }
  }

  function setCosmeticWrapped(id: string, event: Event) {
    setCosmetic(id, (event.target as HTMLInputElement).checked);
  }

  // These are cosmetics the developer gave to their friends with specific Steam IDs
  // These mean something to the people who have them - don't be an asshole
  let blockedCosmetics = [
    "title_stupididiotbaby",
    "title_imnormal",
    "title_bipedalanimaldrawer",
    "pcolor_west",
    "title_lamedev",
    "title_lamedev_real",
    "scolor_midnight_special",
    "title_admiral",
    "title_candy",
    "title_hazard",
    "title_igloo",
    "title_nitrousoxide",
    "title_specialforces",
    "title_zed"
  ];

  const filteredCosmetics = Object.fromEntries(
    Object.entries(cosmetics).filter(([id, cosmetic]) => !blockedCosmetics.includes(id))
  );
</script>

<Section title="Cosmetics">
  <div class="cosmeticsGrid">
    {#each Object.entries(filteredCosmetics) as [id, cosmetic]}
      <div class="cosmeticItem">
        <input
          type="checkbox"
          id={`cosmetic-${id}`}
          checked={save.value.cosmetics_unlocked.value.find((i) => i.value === id) != null}
          on:change={(e) => setCosmeticWrapped(id, e)}
        />
        <img src={`${iconsDir}/${cosmetic.icon}`} alt={cosmetic.name} class="icon" />
        <label for={`cosmetic-${id}`}>{cosmetic.name}</label>
      </div>
    {/each}
  </div>
</Section>

<style>
  .cosmeticsGrid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: var(--spacing-sm);
  }

  .cosmeticItem {
    display: flex;
    align-items: center;
    gap: var(--spacing-sm);
    padding: var(--spacing-xs);
    background-color: rgba(42, 42, 42, 0.5);
    border: 2px solid var(--color-border);
  }

  .cosmeticItem:hover {
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

  .cosmeticItem label {
    flex: 1;
    margin: 0;
    font-size: var(--font-size-xs);
    cursor: pointer;
  }

  @media (max-width: 768px) {
    .cosmeticsGrid {
      grid-template-columns: 1fr;
    }
  }
</style>
