<script>
  import { fly } from 'svelte/transition'
</script>

<div in:fly="{{ x: 200, duration: 200 }}" out:fly="{{ x: -200, duration: 200}}">
  <slot></slot>
</div>

<style>
  div {
    position: absolute
  }
</style>