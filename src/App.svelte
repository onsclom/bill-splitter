<script>
  const states = {
    People: 0,
    Subtotal: 1,
    Items: 2,
    Tip: 3,
    Done: 4,
  };

  let people = [];
  let state = states["People"];
  let curName;
  let subtotal;
  let itemValue;
  $: someoneIsChecked = people.some((people) => people.checked);
  let items = [];
  $: itemTotal = items.reduce((a, b) => a + b.price, 0);
  $: remainingSubtotal = +subtotal - itemTotal;
  let taxPercentage = "6.825";
  let tipPercentage = "20";
  let tipTotal;
  let taxTotal;
  let nameInput;
  let itemPriceInput;

  function setTipFromDollar(dollar) {
    tipTotal = dollar;
    tipPercentage = ((dollar / +subtotal) * 100).toFixed(2);
  }

  function setTaxFromDollar(dollar) {
    taxTotal = dollar;
    taxPercentage = ((dollar / +subtotal) * 100).toFixed(3);
  }

  function setTipFromPercent(percent) {
    tipTotal = (+subtotal * (percent / 100)).toFixed(2);
    tipPercentage = percent;
  }

  function setTaxFromPercent(percent) {
    taxTotal = (+subtotal * (percent / 100)).toFixed(2);
    taxPercentage = percent;
  }

  function updateTaxAndTipFromPercent() {
    setTaxFromPercent(taxPercentage);
    setTipFromPercent(tipPercentage);
  }

  function nextState() {
    state += 1;
    if (state == states.Tip) updateTaxAndTipFromPercent();
  }

  function goBack() {
    state -= 1;
  }

  function submitPerson() {
    people.push({
      name: curName,
      checked: false,
      items: [],
    });
    people = people;
    curName = "";
    nameInput.focus();
  }

  function addItem() {
    items.push({
      price: +itemValue,
      people: people
        .filter((person) => person.checked)
        .map((person) => person.name),
    });
    items = items;
    itemValue = "";
    people.forEach((person) => (person.checked = false));
    people = people;
    itemPriceInput.focus();
  }

  function splitRemaining() {
    items.push({
      price: remainingSubtotal,
      people: people.map((person) => person.name),
    });
    items = items;
  }

  function formatMoney(money) {
    return `$${(+money).toFixed(2)}`;
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
  <button disabled={state == states.People} on:click={goBack}>back</button>
  <br />
  {#if state == states.People}
    <form on:submit|preventDefault={submitPerson} autocomplete="off">
      <label for="name-input">name #{people.length + 1}</label>
      <input
        autofocus
        bind:this={nameInput}
        id="name-input"
        bind:value={curName}
        placeholder="name here"
      />
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
      {#each [...people].reverse() as person}
        <li>
          {person.name}
          <button type="button" on:click={() => removePerson(person.name)}
            >remove</button
          >
        </li>
      {/each}
    </ul>
  {:else if state == states.Subtotal}
    <p>Enter subtotal <em>(total before tax and tip)</em></p>
    <form on:submit|preventDefault={nextState} autocomplete="off">
      <span>$</span>
      <input
        autofocus
        placeholder="0.00"
        class="numberInput"
        inputmode="decimal"
        bind:value={subtotal}
      />
      <input type="submit" disabled={!+subtotal} value="next" />
    </form>
  {:else if state == states.Items}
    {#if +remainingSubtotal.toFixed(2) != 0}
      {#if items.length}
        <button on:click={splitRemaining}
          >split remaining {formatMoney(remainingSubtotal)} equally</button
        >
      {/if}
      <form on:submit|preventDefault={addItem} autocomplete="off">
        <label for="item-price-input">item #{items.length + 1} costs $</label>
        <input
          autofocus
          id="item-price-input"
          placeholder="0.00"
          class="numberInput"
          inputmode="decimal"
          bind:this={itemPriceInput}
          bind:value={itemValue}
        />
        {#if +itemValue > remainingSubtotal}
          <p class="error">
            This item costs more than your remaining subtotal!
          </p>
        {/if}
        <div>this item applies to:</div>
        <div>
          {#each people as person}
            <input
              id="{person.name}-checkbox"
              type="checkbox"
              bind:checked={person.checked}
            />
            <label for="{person.name}-checkbox"
              >{person.name}
              {#if person.checked}
                {`(+${formatMoney(
                  +itemValue / people.filter((p) => p.checked).length
                )})`}
              {/if}
            </label>
            <br />
          {/each}
        </div>
        <button
          on:click|preventDefault={selectAll}
          disabled={people.every((person) => person.checked)}>select all</button
        >
        <input
          type="submit"
          disabled={!someoneIsChecked ||
            !+itemValue ||
            +itemValue > remainingSubtotal}
          value="add item"
        />
      </form>
    {:else}
      <p>
        Nice! The items you added perfectly sum up.
      </p>
      <button on:click={nextState}>next</button>
    {/if}
    {#if items.length}
      <hr />
      <p>
        Remaining subtotal: <strong>{formatMoney(remainingSubtotal)}</strong>
      </p>
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
  {:else if state == states.Tip}
    <form on:submit|preventDefault={nextState} autocomplete="off">
      <label for="tax-input">tax %</label>
      <input
        id="tax-input"
        inputmode="decimal"
        value={taxPercentage}
        placeholder="0.00"
        class="numberInput"
        on:focus={event => event.target.select()}
        on:input={(e) => setTaxFromPercent(e.target.value)}
      />
      =
      <label for="tax-input-dollar">$</label>
      <input
        id="tax-input-dollar"
        inputmode="decimal"
        value={taxTotal}
        placeholder="0.00"
        class="numberInput"
        on:focus={event => event.target.select()}
        on:input={(e) => setTaxFromDollar(e.target.value)}
      />
      <br />
      <label for="tip-input">tip %</label>
      <input
        id="tip-input"
        inputmode="decimal"
        value={tipPercentage}
        placeholder="0.00"
        class="numberInput"
        on:focus={event => event.target.select()}
        on:input={(e) => setTipFromPercent(e.target.value)}
      />
      =
      <label for="tax-input-dollar">$</label>
      <input
        id="tip-input-dollar"
        inputmode="decimal"
        value={tipTotal}
        placeholder="0.00"
        class="numberInput"
        on:focus={event => event.target.select()}
        on:input={(e) => setTipFromDollar(e.target.value)}
      />
      <br />
      <input
        disabled={+tipPercentage === NaN || +taxPercentage === NaN}
        type="submit"
        value="next"
        on:click={nextState}
      />
    </form>
    <hr>
    <div>Subtotal: {formatMoney(+subtotal)}</div>
    <div>Tax: {formatMoney(taxTotal)}</div>
    <div>Tip: {formatMoney(tipTotal)}</div>
    <div>Total: {formatMoney(+subtotal + +taxTotal + +tipTotal)}</div>
  {:else if state == states.Done}
    <ul>
      {#each people as person}
        <li>
          {person.name}: {formatMoney(
            addTaxAndTip(
              items
                .filter((item) => item.people.includes(person.name))
                .reduce((prev, cur) => prev + cur.price / cur.people.length, 0)
            )
          )}
          <!-- <ul>
            {#each items as item}
              {#if item.people.includes(person.name)}
                <li>
                  {formatMoney(item.price / item.people.length)}
                </li>
              {/if}
            {/each}
          </ul> -->
        </li>
      {/each}
    </ul>
    <hr>
    <div>Subtotal: {formatMoney(+subtotal)}</div>
    <div>Tax: {formatMoney(taxTotal)}</div>
    <div>Tip: {formatMoney(tipTotal)}</div>
    <div>Total: {formatMoney(+subtotal + +taxTotal + +tipTotal)}</div>
  {/if}
</main>

<style>
  ul {
    margin: 0;
  }
  .error {
    color: #e22;
  }
  .numberInput {
    max-width: 4rem;
  }
</style>
