<script lang="ts">
  type TOptions = "decode" | "encode";
  type TSave = {
    id: number;
    title: string;
    input: string;
    output: string;
    type: TOptions;
  };
  import { Base64 } from "js-base64";

  const invalid = "Invalid input!";

  let saves: TSave[] = getSaves();

  function getSaves() {
    let ls = localStorage.getItem("saves");
    if (!ls) {
      return [];
    }

    return JSON.parse(ls) as TSave[];
  }

  function setSaves(saves: TSave[]) {
    localStorage.setItem("saves", JSON.stringify(saves));
  }

  function addSave(save: TSave) {
    saves = [...saves, save];
    setSaves(saves);
  }

  function deleteSave(id: number) {
    saves = saves.filter((save) => save.id !== id);
    setSaves(saves);
  }

  function editSaveTitle(id: number, value: string) {
    let idx = saves.findIndex((s) => s.id === id);
    saves[idx].title = value;
    saves = saves;
    setSaves(saves);
    editingIdx = null;
    editingInput = "";
  }

  let options: TOptions = "decode";
  let URLSave: boolean = false;
  let SeparateLine: boolean = false;
  let editingIdx: null | number = null;
  let editingInput: string = "";
  let input = "";
  let output = "";
  let showToast = false;

  function decodeInput(value: string) {
    let ret = "";
    if (value === "") {
      return value;
    }
    try {
      ret = Base64.decode(value);
    } catch {
      ret = invalid;
    }
    return ret;
  }

  function encodeInputSeparately(value: string) {
    let valSplit = value.split("\n");
    let encodedVals = valSplit.map((val) => encodeInput(val));
    return encodedVals.join("\n");
  }

  function encodeInput(value: string) {
    if (value === "") {
      return "";
    }
    if (URLSave) {
      return Base64.encodeURL(value);
    }
    return Base64.encode(value);
  }

  function processInput(value: string) {
    if (options === "encode") {
      if (SeparateLine) {
        return encodeInputSeparately(value);
      }
      return encodeInput(value);
    } else if (options === "decode") {
      return decodeInput(value);
    }
    return "";
  }

  function changeOptions(value: string) {
    options = value as TOptions;
    output = processInput(input);
  }

  function handleCardClick(id: number) {
    let save = saves.filter((s) => s.id === id)[0];
    input = save.input;
    output = save.output;
    options = save.type;
    scrollTo(0, 0);
  }

  function handleSaveButton() {
    const lastSave = saves.at(saves.length - 1);
    let lastId = 0;
    if (lastSave) {
      lastId = lastSave.id;
    }
    const save: TSave = {
      id: lastId + 1,
      title: `${new Date(Date.now()).toUTCString()}`,
      input: input,
      output: output,
      type: options,
    };
    addSave(save);
  }

  function handleDeleteButton(id: number) {
    deleteSave(id);
  }

  function handleCopyButton(value: string) {
    navigator.clipboard.writeText(value);
    handleShowToast();
  }

  function handleShowToast() {
    showToast = true;
    setTimeout(() => (showToast = false), 3000);
  }

  $: output = processInput(input);
</script>

<main>
  <section class="container mx-auto lg:px-8">
    <div class="flex justify-center m-4">
      <h1 class="text-2xl text-center transition-all md:text-4xl">
        Base64 <span
          class={options === "decode"
            ? "underline decoration-accent"
            : "line-through"}>Decode</span
        >
        &
        <span
          class={options === "encode"
            ? "underline decoration-accent"
            : "line-through"}>Encoder</span
        >
      </h1>
    </div>
    <div class="flex justify-center m-4">
      <p class="text-center">
        We never store your data anywhere on the internet, it's all saved on
        your device. <br />Only works with UTF-8 (for now).
      </p>
    </div>
    <section class="flex justify-center lg:m-4">
      <div class="join">
        <input
          class="join-item btn"
          type="radio"
          name="options"
          aria-label="Decode"
          value="decode"
          on:change={() => changeOptions("decode")}
          checked={options === "decode"}
        />
        <input
          class="join-item btn"
          type="radio"
          name="options"
          aria-label="Encode"
          value="encode"
          on:change={() => changeOptions("encode")}
          checked={options === "encode"}
        />
      </div>
    </section>

    <div class="flex justify-center m-4">
      {#if options === "encode"}
        <div class="mr-8 form-control">
          <label class="cursor-pointer label">
            <span class="label-text">URL-Safe Encoding</span>
            <input
              type="checkbox"
              class="ml-2 checkbox checkbox-primary"
              checked={URLSave}
              on:change={() => {
                URLSave = !URLSave;
                output = processInput(input);
              }}
            />
          </label>
        </div>
      {/if}
      <div class="form-control">
        <label class="cursor-pointer label">
          <span class="label-text">Separate per Line</span>
          <input
            type="checkbox"
            class="ml-2 checkbox checkbox-primary"
            checked={SeparateLine}
            on:change={() => {
              SeparateLine = !SeparateLine;
              output = processInput(input);
            }}
          />
        </label>
      </div>
    </div>
    <section>
      <div class="flex justify-center m-4 lg:m-6">
        <textarea
          class="max-w-full textarea textarea-primary"
          placeholder="Input your Base64 here..."
          required
          spellcheck="false"
          autocorrect="off"
          bind:value={input}
          rows="8"
          cols="80"
        />
      </div>
      <div class="flex justify-center m-4 lg:m-6">
        <textarea
          class={`max-w-full textarea ${
            output === invalid ? "textarea-error" : "textarea-secondary"
          }`}
          placeholder="Output will go here"
          readonly
          spellcheck="false"
          autocorrect="off"
          bind:value={output}
          rows="8"
          cols="80"
        />
      </div>
      <div class="flex justify-center mb-4">
        <button class="btn btn-primary" on:click={handleSaveButton}>Save</button
        >
      </div>
    </section>
    <section class="flex flex-col-reverse items-center justify-center">
      {#each saves as fav (fav.id)}
        <div
          class={`w-11/12 mb-4 shadow-xl md:w-9/12 card ${
            fav.type === "decode"
              ? "bg-primary text-primary-content"
              : "bg-accent text-accent-content"
          }  `}
        >
          <div class="card-body">
            <div class="flex flex-row-reverse justify-between">
              {#if editingIdx === fav.id}
                <div class="flex">
                  <input
                    type="text"
                    class="w-full max-w-xs input input-bordered bg-slate-100"
                    bind:value={editingInput}
                  />
                  <button
                    class="btn btn-accent"
                    on:click={() => editSaveTitle(fav.id, editingInput)}
                    >OK</button
                  >
                </div>
              {:else}
                <div
                  class="flex flex-col items-center md:justify-end md:flex-row card-actions"
                >
                  <p class="mr-2 text-sm uppercase">{fav.type}</p>
                  <button
                    class="btn btn-square btn-sm"
                    on:click={() => {
                      editingIdx = fav.id;
                      editingInput = fav.title;
                    }}>✏️</button
                  >
                  <button
                    class="btn btn-square btn-sm"
                    on:click={() => handleDeleteButton(fav.id)}>🗑️</button
                  >
                </div>
                <h2 class="card-title">{fav.title}</h2>
              {/if}
            </div>
            <h3 class="text-lg underline">Input</h3>
            <div class="relative">
              <button
                class="absolute top-1 right-1 btn btn-square btn-info btn-sm"
                on:click={() => handleCopyButton(fav.input)}>📄</button
              >
              <textarea
                readonly
                class="w-full rounded-md textarea textarea-primary bg-slate-100"
                >{fav.input}</textarea
              >
            </div>

            <h3 class="text-lg underline">Output</h3>
            <div class="relative">
              <button
                class="absolute top-1 right-1 btn btn-square btn-info btn-sm"
                on:click={() => handleCopyButton(fav.output)}>📄</button
              >
              <textarea
                readonly
                class="w-full rounded-md textarea textarea-secondary bg-slate-100"
                >{fav.output}</textarea
              >
            </div>
          </div>
        </div>
      {/each}
    </section>
  </section>
  {#if showToast}
    <div class="toast toast-end">
      <div class="alert alert-success">
        <span>Text copied succesfully!</span>
      </div>
    </div>
  {/if}
  <footer>
    Made by Rizdar, with sweat and tears and time and no sleep and anything and
    everything.
  </footer>
</main>

<style>
</style>
