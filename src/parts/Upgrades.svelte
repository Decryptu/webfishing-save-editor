<script lang="ts">
  import type { WebfishingSave } from "../game/types";
  import Section from "../components/Section.svelte";

  export let save: WebfishingSave;
</script>

<Section title="Upgrades">
  <fieldset class="grid">
    <div>
      <label for="loanLevel">Loan level</label>
      <input type="number" id="loanLevel" bind:value={save.value.loan_level.value} />
    </div>

    <div>
      <label for="loanLeft">Loan left</label>
      <input type="number" id="loanLeft" bind:value={save.value.loan_left.value} />
    </div>

    <div>
      <label for="maxBait">Max bait</label>
      <input type="number" id="maxBait" bind:value={save.value.max_bait.value} />
    </div>

    <div>
      <label for="buddyLevel">Buddy level</label>
      <input type="number" id="buddyLevel" bind:value={save.value.buddy_level.value} />
    </div>

    <div>
      <label for="buddySpeed">Buddy speed</label>
      <input type="number" id="buddySpeed" bind:value={save.value.buddy_speed.value} />
    </div>

    <div>
      <label for="rodChance">Rod chance</label>
      <input type="number" id="rodChance" bind:value={save.value.rod_chance.value} />
    </div>

    <div>
      <label for="rodLuck">Rod luck</label>
      <input type="number" id="rodLuck" bind:value={save.value.rod_luck.value} />
    </div>

    <div>
      <label for="rodPower">Rod power</label>
      <input type="number" id="rodPower" bind:value={save.value.rod_power.value} />
    </div>

    <div>
      <label for="rodSpeed">Rod speed</label>
      <input type="number" id="rodSpeed" bind:value={save.value.rod_speed.value} />
    </div>
  </fieldset>
</Section>
