<script>
  // done, current, todo
  export let state = "current" 
</script>

<div class="{state}">
</div>

<style>
  div {
    align-self: center;
    box-sizing: border-box;
    width: 1rem;
    height: 1rem;
    display: inline-block;
    margin: .2rem;
    border-radius: 100%;
    transition: all .4s ease-out;
    border: solid black 2px;
  }

  .current {
    background-color: black;
  }

  .done {
    background-color: gray;
  }

  .todo {
    background-color: white;
  }
</style>