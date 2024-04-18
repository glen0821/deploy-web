<svelte:options accessors />

<script>
  export let PLACEHOLDER = "Overide placeholder";
  export let BINDTHIS = "";
  export let TITLE = "Overide title";
  export let TYPE = "text";
  export let COLOR = "black";
  export let ID = "";
  export let ONCLICK =()=>{};
</script>

{#if TYPE === "text"}
  <p class="p-2 font-bold text-{COLOR}">{TITLE}</p>
  <input
    type="text"
    placeholder={PLACEHOLDER}
    bind:value={BINDTHIS}
    class="w-full h-10 bg-slate-100 p-2 focus:outline-none"
  />
{:else if TYPE === "email"}
  <p class="p-2 font-bold text-{COLOR}">{TITLE}</p>
  <input
    type="email"
    placeholder={PLACEHOLDER}
    bind:value={BINDTHIS}
    class="w-full h-10 bg-slate-100 p-2 focus:outline-none"
  />
{:else if TYPE === "password"}
  <p class="p-2 font-bold text-{COLOR}">{TITLE}</p>
  <input
    type="password"
    placeholder={PLACEHOLDER}
    bind:value={BINDTHIS}
    class="w-full h-10 bg-slate-100 p-2 focus:outline-none"
  />
{:else if TYPE === "number"}
  <p class="p-2 font-bold text-{COLOR}">{TITLE}</p>
  <input
    type="number"
    placeholder={PLACEHOLDER}
    bind:value={BINDTHIS}
    class="w-full h-10 bg-slate-100 p-2 focus:outline-none"
  />
{:else if TYPE === "date"}
  <p class="p-2 font-bold text-{COLOR}">{TITLE}</p>
  <input
    type="date"
    bind:value={BINDTHIS}
    id={ID}
    on:click={ONCLICK}
    class="w-full h-10 bg-slate-100 p-2 focus:outline-none"
  />
{:else if TYPE === "cordion"}
  <p class="p-2 font-bold text-{COLOR}">{TITLE}</p>
  <select
    class="w-full h-10 bg-slate-100 p-2 focus:outline-none"
    bind:value={BINDTHIS}
  >
    <option value="Male">Male</option>
    <option value="Female">Female</option>
  </select>
  {:else if TYPE === "off-positions"}
  <p class="p-2 font-bold text-{COLOR}">{TITLE}</p>
  <select
    class="w-full h-10 bg-slate-100 p-2 focus:outline-none"
    bind:value={BINDTHIS}
  >
    <option value="Captain">Captain</option>
    <option value="Administrator">Administrator</option>
    <option value="Executive Officer">Executive Officer</option>
    <option value="Secretary">Secretary</option>
    <option value="Treasurer">Treasurer</option>
    <option value="Councilor">Councilor</option>
    <option value="Purok Leader">Purok Leader</option>
  </select>
  {:else if TYPE === "file"}
  <p class="p-2 font-bold text-{COLOR}">{TITLE}</p>
  <input
    type="file"
    accept="image/*"
    on:change={(event) => BINDTHIS = event.target.files[0]}
    class="w-full h-10 bg-slate-100 p-2 focus:outline-none"
    id={ID}
  />
{/if}
