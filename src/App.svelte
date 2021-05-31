<script lang="ts">
  import { onMount } from 'svelte'

  import Dropdown from './Dropdown.svelte'

  let categories: string[] = []
  let category = ''
  let units: { [key: string]: string } = {}

  let fromUnit = { name: '', symbol: '' }
  let toUnit = { name: '', symbol: '' }

  let amount: number

  let result: number

  const setSymbol = (
    e: Event & {
      currentTarget: EventTarget & HTMLSelectElement
    },
    obj: Required<typeof fromUnit>
  ) => {
    const unitName = e.currentTarget.value

    if (!units || !unitName) return

    const unitSymbol = Object.entries(units).filter(
      ([_symbol, name]) => name === unitName
    )[0][0]

    obj.symbol = unitSymbol

    if (amount > 0) {
      handleInput()
    }
  }

  onMount(async () => {
    const res = await fetch(`${API_URL}/categories`)

    const data: { categories: string[] } = await res.json()

    categories = data.categories.map(category => category.replace(/_/g, ' '))

    category = categories[0]
  })

  $: {
    async function getUnitsForCategory() {
      if (!category) return

      const res = await fetch(
        `${API_URL}/categories/${category.replace(/ /g, '_')}`
      )

      const data = await res.json()

      units = data

      if (!data) return

      const keys = Object.keys(data)

      fromUnit = { symbol: keys[0], name: data[keys[0]] }
      toUnit = { symbol: keys[1], name: data[keys[1]] }
    }

    getUnitsForCategory()
  }

  async function handleInput() {
    if (amount <= 0 || fromUnit.name === toUnit.name) return

    const res = await fetch(
      `${API_URL}/categories/${category.replace(/ /g, '_')}?from=${
        fromUnit.symbol
      }&to=${toUnit.symbol}&amount=${amount}`
    )

    if (res.status !== 200) {
      console.error(res.statusText)

      return
    }

    const data: { result: number } = await res.json()

    result = data.result
  }
</script>

<main>
  <h1>Convertify</h1>
  <div class="root-div">
    <Dropdown
      name="category"
      options={categories}
      bind:currentValue={category}
      title="Category"
      onChange={undefined}
    />
    <Dropdown
      name="from_unit"
      options={Object.values(units)}
      bind:currentValue={fromUnit.name}
      title="Convert From"
      onChange={e => setSymbol(e, fromUnit)}
    />
    <Dropdown
      name="to_unit"
      options={Object.values(units).filter(unit => fromUnit.name !== unit)}
      bind:currentValue={toUnit.name}
      title="Convert To"
      onChange={e => setSymbol(e, toUnit)}
    />
    <label for="amount">
      <input
        type="number"
        name="amount"
        id="amount"
        placeholder="Amount"
        min="0"
        bind:value={amount}
        on:input={e => (!e.currentTarget.value ? undefined : handleInput())}
      />
      <span class="field-label">Amount</span>
    </label>
    <div class="result-wrapper">
      <span class="equal-sign">=</span>
      <span class="result">{result ?? ''}</span>
    </div>
  </div>
</main>
<footer>
  <div>
    <span>©{new Date().getFullYear()} |{' '}</span>
    <a href="https://ulises.codes" target="_blank" rel="noopener author"
      >Ulises Himely</a
    >
  </div>
</footer>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 720px;
    margin: 0 auto;
  }

  h1 {
    color: var(--red);
    text-transform: uppercase;
    font-size: 2.5rem;
    font-weight: 100;
  }

  .root-div {
    margin: auto;
    max-width: 540px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    gap: 32px;
  }

  .result-wrapper {
    height: 75px;
    box-shadow: var(--shadow);
    border-radius: 12px;
    display: flex;
    align-items: center;
    justify-content: space-around;
  }

  .equal-sign {
    flex-grow: 0.4;
    text-align: right;
  }

  .result {
    flex: 0.5 0 75px;
    text-align: left;
    font-weight: bold;
  }

  footer {
    position: fixed;
    bottom: 0;
    height: 75px;
    width: 100%;
  }

  footer > div {
    margin: auto;
    max-width: 720px;
    text-align: center;
  }
</style>
