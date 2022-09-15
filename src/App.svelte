<script>
  let people = [];
  let state = "People";
  let curName;
  let subtotalString;
  $: subtotal = Number(subtotalString);
  let itemValueString;
  $: itemValue = Number(itemValueString);
  $: someoneIsChecked = people.some((people) => people.checked);
  let items = [];
  let shareItem = false;
  $: itemTotal = items.reduce((a, b) => a + b.price, 0);
  $: remainingSubtotal = subtotal - itemTotal;
  let selectedName;
  let taxPercentage = "6.825";
  let tipPercentage = "20";
  let tipTotal;
  let taxTotal;

  function setTipFromDollar(dollar) {
    tipTotal = dollar;
    tipPercentage = ((dollar / subtotal) * 100).toFixed(2);
  }

  function setTaxFromDollar(dollar) {
    taxTotal = dollar;
    taxPercentage = ((dollar / subtotal) * 100).toFixed(3);
  }

  function setTipFromPercent(percent) {
    tipTotal = (subtotal * (percent / 100)).toFixed(2);
    tipPercentage = percent;
  }

  function setTaxFromPercent(percent) {
    taxTotal = (subtotal * (percent / 100)).toFixed(2);
    taxPercentage = percent;
  }

  function updateTaxAndTipFromPercent() {
    setTaxFromPercent(taxPercentage);
    setTipFromPercent(tipPercentage);
  }

  function nextState() {
    if (state == "People") state = "Subtotal";
    else if (state == "Subtotal") state = "Items";
    else if (state == "Items") {
      updateTaxAndTipFromPercent();
      state = "Tip";
    } else if (state == "Tip") state = "Done";
  }

  function goBack() {
    if (state == "Subtotal") state = "People";
    else if (state == "Items") {
      updateTaxAndTipFromPercent();
      state = "Subtotal";
    } else if (state == "Tip") state = "Items";
    else if (state == "Done") state = "Tip";
  }

  function submitPerson() {
    people.push({
      name: curName,
      checked: false,
      items: [],
    });
    people = people;
    curName = "";
  }

  function addItem() {
    items.push({
      price: itemValue,
      people: shareItem
        ? people.filter((person) => person.checked).map((person) => person.name)
        : [selectedName],
    });
    items = items;
    itemValueString = "";
    shareItem = false;
    selectedName = "";
    people.forEach((person) => (person.checked = false));
    people = people;
  }

  function splitRemaining() {
    items.push({
      price: remainingSubtotal,
      people: people.map((person) => person.name),
    });
    items = items;
  }

  function formatMoney(money) {
    return `$${Number(money).toFixed(2)}`;
  }

  function selectAll() {
    people.forEach((person) => (person.checked = true));
    people = people;
  }

  function addTaxAndTip(money) {
    return (
      money + money * (+taxPercentage / 100) + money * (+tipPercentage / 100)
    );
  }

  function removePerson(name) {
    people = people.filter((person) => person.name != name);
    items = items.filter((item) => !item.people.includes(name));
  }

  function removeItem(removedItem) {
    items = items.filter((item) => item != removedItem);
  }
</script>

<main>
  <button disabled={state == "People"} on:click={goBack}>back</button>
  <br />
  {#if state == "People"}
    <form on:submit|preventDefault={submitPerson}>
      <label for="name-input">name #{people.length + 1}</label>
      <input id="name-input" bind:value={curName} placeholder="name here" />
      <br />
      <input type="submit" disabled={!curName} value="add name" />
    </form>
    <button on:click={nextState} disabled={people.length < 2}>next</button>
    {#if people.length}
      <hr />
    {/if}
    {#if people.length >= 2}
      <div>{people.length} people named</div>
    {/if}
    <ul id="name-ul">
      {#each people as person}
        <li>
          {person.name}
          <button type="button" on:click={() => removePerson(person.name)}
            >remove</button
          >
        </li>
      {/each}
    </ul>
  {:else if state == "Subtotal"}
    <p>Enter subtotal <em>(total before tax and tip)</em></p>
    <form on:submit|preventDefault={nextState}>
      <span>$</span>
      <input
        placeholder="subtotal"
        inputmode="decimal"
        bind:value={subtotalString}
      />
      <input type="submit" disabled={!subtotal} value="next" />
    </form>
  {:else if state == "Items"}
    {#if +remainingSubtotal.toFixed(2) != 0}
      {#if items.length}
        <button on:click={splitRemaining}>split remaining equally</button>
        <p>OR</p>
      {/if}
      <form on:submit|preventDefault={addItem}>
        <label for="item-price-input">item #{items.length + 1} costs $</label>
        <input
          id="item-price-input"
          placeholder="item price"
          inputmode="decimal"
          bind:value={itemValueString}
        />
        <br />
        <input
          id="split-item-checkbox"
          type="checkbox"
          disabled={!itemValue}
          bind:checked={shareItem}
        />
        <label for="split-item-checkbox">people shared this item</label>
        <div>
          {#if shareItem}
            <div>this item applies to:</div>
            <div>
              {#each people as person}
                <input
                  id="{person.name}-checkbox"
                  type="checkbox"
                  bind:checked={person.checked}
                />
                <label for="{person.name}-checkbox">{person.name}</label>
                <br />
              {/each}
            </div>
            <button
              on:click|preventDefault={selectAll}
              disabled={people.every((person) => person.checked)}
              >select all</button
            >
          {:else}
            <label for="person-select">this item applies to</label>
            <select disabled={!itemValue} bind:value={selectedName}>
              <option value="" disabled selected>select name</option>
              {#each people as person}
                <option value={person.name}>{person.name}</option>
              {/each}
            </select>
          {/if}
        </div>
        <input
          type="submit"
          disabled={(shareItem && !someoneIsChecked) ||
            (!shareItem && !selectedName) ||
            !itemValue}
          value="add item"
        />
      </form>
    {:else}
      <button on:click={nextState}>next</button>
    {/if}
    {#if items.length}
      <hr />
    {/if}
    {#if items.length}
      <p>
        Remaining subtotal: <strong>{formatMoney(remainingSubtotal)}</strong>
      </p>
      <div>{items.length} items enterred</div>
    {/if}
    {#each items as item}
      <ul>
        <li>
          {formatMoney(item.price)} - {item.people.join(", ")}
          <button
            on:click={() => {
              removeItem(item);
            }}>remove</button
          >
        </li>
      </ul>
    {/each}
  {:else if state == "Tip"}
    <form on:submit|preventDefault={nextState}>
      <label for="tax-input">tax %</label>
      <input
        id="tax-input"
        inputmode="decimal"
        value={taxPercentage}
        on:input={(e) => setTaxFromPercent(e.target.value)}
      />
      =
      <label for="tax-input-dollar"
        >$<label>
          <input
            id="tax-input-dollar"
            inputmode="decimal"
            value={taxTotal}
            on:input={(e) => setTaxFromDollar(e.target.value)}
          />
          <br />
          <label for="tip-input">tip %</label>
          <input
            id="tip-input"
            inputmode="decimal"
            value={tipPercentage}
            on:input={(e) => setTipFromPercent(e.target.value)}
          />
          =
          <label for="tax-input-dollar"
            >$<label>
              <input
                id="tip-input-dollar"
                inputmode="decimal"
                value={tipTotal}
                on:input={(e) => setTipFromDollar(e.target.value)}
              />
            </label>
            <br />
            <input
              disabled={+tipPercentage === NaN || +taxPercentage === NaN}
              type="submit"
              value="next"
              on:click={nextState}
            />
          </label></label
        ></label
      >
    </form>
  {:else if state == "Done"}
    <ul>
      {#each people as person}
        <li>
          {person.name} owes {formatMoney(
            addTaxAndTip(
              items
                .filter((item) => item.people.includes(person.name))
                .reduce((prev, cur) => prev + cur.price / cur.people.length, 0)
            )
          )}
          <ul>
            {#each items as item}
              {#if item.people.includes(person.name)}
                <li>
                  {formatMoney(item.price / item.people.length)}
                </li>
              {/if}
            {/each}
          </ul>
        </li>
      {/each}
    </ul>

    <div>Subtotal: {formatMoney(subtotal)}</div>
    <div>Tax: {formatMoney(taxTotal)}</div>
    <div>Tip: {formatMoney(tipTotal)}</div>
    <div>Total: {formatMoney(Number(subtotal) + Number(taxTotal) + Number(tipTotal))}</div>

    <button>Reset</button>
  {/if}
</main>

<style>
  ul {
    margin: 0;
  }
</style>
