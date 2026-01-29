<script lang="ts">
  import Editor from "./Editor.svelte";
  import type { WebfishingSave } from "./game/types";
  import { readFile, BinaryReader, BinaryWriter, writeFile } from "./lib/binary";
  import { readVariant, writeVariant } from "./lib/godot";
  import type { GodotVariant } from "./lib/types";

  let save: WebfishingSave | null;
  let parseError: string | null = null;

  async function uploadSave(e: Event) {
    const target = e.target as HTMLInputElement;
    const file = target.files?.[0];
    if (file != null) {
      parseError = null;
      save = null;
      try {
        const bytes = await readFile(file);
        const reader = new BinaryReader(bytes);
        reader.readUInt32(); // Size, which we don't care about
        save = readVariant(reader) as unknown as WebfishingSave;
        console.log(save);
      } catch (e) {
        console.error("Failed to parse save file:", e);
        parseError = e instanceof Error ? e.message : String(e);
      }
    }
  }

  function downloadSave() {
    const writer = new BinaryWriter();
    writeVariant(writer, save as unknown as GodotVariant);
    const bytes = writer.toBytes();

    const writer2 = new BinaryWriter();
    writer2.writeUInt32(bytes.length);
    writer2.write(bytes);

    writeFile(writer2.toBytes(), "webfishing_save_slot_0.sav");
  }
</script>

<hgroup>
  <h1>WEBFISHING Save Editor</h1>
  <p>
    <a href="https://github.com/Decryptu/webfishing-save-editor">GitHub</a> - maintained by
    <a href="https://github.com/Decryptu">Decryptu</a>
  </p>
</hgroup>

<p>
  Your save is located at <code>%AppData%\Godot\app_userdata\webfishing_2_newver\webfishing_save_slot_0.sav</code>. Back
  up your save before editing it!
</p>

<details>
  <summary>Important notes</summary>
  <ul>
    <li><strong>Close the game</strong> before replacing your save file, otherwise the game will overwrite your changes when it saves.</li>
    <li><strong>Disable Steam Cloud sync</strong> for WEBFISHING if your edits keep reverting. Steam may restore the old save on launch.
      (Steam &rarr; Right-click WEBFISHING &rarr; Properties &rarr; General &rarr; uncheck "Keep game saves in the Steam Cloud")</li>
    <li>Both <code>.sav</code> and <code>.backup</code> files are supported.</li>
  </ul>
</details>

<div class="fileUpload" role="group">
  <input type="file" name="file" accept=".sav,.backup" on:change={uploadSave} />

  {#if save}
    <input type="button" value="Download save" on:click={downloadSave} />
  {/if}
</div>

{#if parseError}
  <article class="parseError">
    <header>Failed to parse save file</header>
    <p>The file could not be read as a WEBFISHING save. This can happen if the file is corrupted, from an unsupported game version, or not a save file.</p>
    <pre><code>{parseError}</code></pre>
    <p><small>Tip: if uploading <code>.sav</code> doesn't work, try the <code>.backup</code> file from the same folder.</small></p>
  </article>
{/if}

{#if save}
  <div id="editor">
    <Editor {save} />
  </div>
{/if}

