<script lang="ts">
  export let options: string[]
  export let name: string
  export let currentValue: string
  export let title: string
  export let onChange: svelte.JSX.EventHandler<Event, HTMLSelectElement>
</script>

<label for={name}>
  <select
    {name}
    id={name}
    bind:value={currentValue}
    on:blur={onChange}
    on:change={e => e.currentTarget.blur()}
  >
    {#each options as option}
      <option value={option}>{option}</option>
    {/each}
  </select>
  <span class="field-label">{title}</span>
</label>

<style>
  select {
    text-transform: capitalize;
  }
</style>
