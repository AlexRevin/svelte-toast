<script>
import { fade, fly } from 'svelte/transition'
import { flip } from 'svelte/animate'
import { toast } from './stores.js'
import ToastItem from './ToastItem.svelte'

export let options = {}
export let target = 'default'
export let wrapper

$: toast._init(target, options)

let items
$: items = $toast.filter(i => i.target === target)

const getCss = theme => Object.keys(theme).reduce((a, c) => `${a}${c}:${theme[c]};`, '')
</script>

<style>
ul {
  top: var(--toastContainerTop,1.5rem);
  right: var(--toastContainerRight,2rem);
  bottom: var(--toastContainerBottom,auto);
  left: var(--toastContainerLeft,auto);
  position: fixed;
  margin: 0;
  padding: 0;
  list-style-type: none;
  pointer-events: var(--toastContainerPointerEvents, auto);
  z-index: 9999;
}
</style>

<span class={wrapper}>
  <ul>
    {#each items as item (item.id)}
    <li
      in:fly={item.intro}
      out:fade
      animate:flip={{ duration: 200 }}
      style={getCss(item.theme)}
      >
      <ToastItem {item} />
    </li>
    {/each}
  </ul>
</span>