<script>
  import { fly } from 'svelte/transition'
  export let reverse = false
  let distance = 200
  let dur = 300
</script>

<div in:fly="{{ x: (reverse?-distance:distance), duration: dur }}" 
out:fly="{{ x: (reverse?distance:-distance), duration: dur}}">
  <slot></slot>
</div>

<style>
  div {
    position: absolute
  }
</style>