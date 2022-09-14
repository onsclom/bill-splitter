<script>
  let people = [
    { name: "Jeff", checked: false },
    { name: "Bob", checked: false },
    { name: "Joe", checked: false },
  ];
  let state = "People";
  let curName = "";
  let subtotalString = 25.25;
  $: subtotal = Number(subtotalString);
  let itemValueString;
  $: itemValue = Number(itemValueString);
  $: someoneIsChecked = people.some((people) => people.checked);
  let items = [];

  function nextState() {
    if (state == "People") state = "Subtotal";
    else if (state == "Subtotal") state = "Items";
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
    })
    items = items;
    itemValueString=""
  }
</script>

<main>
  {#if state == "People"}
    <form on:submit|preventDefault={submitPerson}>
      <p>Add names</p>
      <input bind:value={curName} placeholder="John Smith" />
      <input type="submit" disabled={!curName} value="add" />
    </form>
    <button on:click={nextState} disabled={people.length < 2}>done</button>
    {#if people.length >= 2}
      <p>{people.length} people added</p>
    {/if}
    <ul>
      {#each people as person}
        <li>{person.name}</li>
      {/each}
    </ul>
  {:else if state == "Subtotal"}
    <p>Enter subtotal (total before tax and tip)</p>
    <form on:submit|preventDefault={nextState}>
      <span>$</span>
      <input
        placeholder="Subtotal"
        inputmode="decimal"
        bind:value={subtotalString}
      />
      <input type="submit" disabled={!subtotal} value="enter" />
    </form>
  {:else if state == "Items"}
    <p>Remaining subtotal: <span>{subtotal}</span></p>

    <form on:submit|preventDefault={addItem}>
      <p>Detail item</p>
      <span>$</span>
      <input
        placeholder="item price"
        inputmode="decimal"
        bind:value={itemValueString}
      />
      <div>
        <p>item applies to:</p>
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
      <input
        type="submit"
        disabled={!someoneIsChecked || !itemValue}
        value="add item"
      />
    </form>

    <p>OR</p>

    <button>split remaining</button>

    {#each items as item}
      <ul>
        {item.price}
      </ul>
    {/each}
  {/if}
</main>

<style>
</style>
