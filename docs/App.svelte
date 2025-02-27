<script>
import { tick } from 'svelte'
import { SvelteToast, toast } from '../src'
import Prism from './Prism.svelte'
import DummyComponent from './Dummy.svelte'
import camelCase from 'camelcase'

// Hoist to `window` for debug
window.toast = toast

let selected
let code = '// Tap a button below'
let colors = false
let bottom = false
let options = {}

const handleClick = btn => {
  selected = btn.name
  code = btn.code
  btn.run()
  gtag('event', 'toast', { event_label: btn.name })
}

const buttons = [
  {
    name: 'DEFAULT',
    code: `toast.push('Hello world!')`, // eslint-disable-line quotes
    run: () => {
      toast.push('Hello world!')
    }
  },
  {
    name: 'GREEN',
    code: `toast.push('Success!', {
  theme: {
    '--toastBackground': '#48BB78',
    '--toastProgressBackground': '#2F855A'
  }
})`,
    run: () => {
      toast.push('Success!', { theme: { '--toastBackground': '#48BB78', '--toastProgressBackground': '#2F855A' } })
    }
  },
  {
    name: 'RED',
    code: `toast.push('Danger!', {
  theme: {
    '--toastBackground': '#F56565',
    '--toastProgressBackground': '#C53030'
  }
})`,
    run: () => {
      toast.push('Danger!', { theme: { '--toastBackground': '#F56565', '--toastProgressBackground': '#C53030' } })
    }
  },
  {
    name: 'RICH HTML',
    code: `toast.push(\`<strong>You won the jackpot!</strong><br>
  Click <a href="#" target="_blank">here</a> for details! 😛\`)`,
    run: () => {
      toast.push('<strong>You won the jackpot!</strong><br>Click <a href="#" target="_blank">here</a> for details! 😛')
    }
  },
  {
    name: 'LONG DURATION',
    code: `toast.push('Watching the paint dry...', { duration: 20000 })`, // eslint-disable-line quotes
    run: () => {
      toast.push('Watching the paint dry...', { duration: 20000 })
    }
  },
  {
    name: 'NON-DISMISSABLE',
    code: `toast.push('Where the close btn?!?', {
  initial: 0,
  next: 0,
  dismissable: false
})`,
    run: () => {
      toast.push('Where the close btn?!?', { initial: 0, dismissable: false })
    }
  },
  {
    name: 'REMOVE LAST TOAST',
    code: `// Remove the latest toast
toast.pop()

// Or remove a particular one
const id = toast.push('Yo!')
toast.pop(id)`,
    run: () => {
      toast.pop()
    }
  },
  {
    name: 'FLIP PROGRESS BAR',
    code: `toast.push('Progress bar is flipped', {
  initial: 0,
  next: 1
})`,
    run: () => {
      toast.push('Progress bar is flipped', { initial: 0, next: 1 })
    }
  },
  {
    name: 'USE AS LOADING INDICATOR',
    code: `const sleep = t => new Promise(resolve => setTimeout(resolve, t))

const id = toast.push('Loading, please wait...', {
  duration: 300,
  initial: 0,
  next: 0,
  dismissable: false
})

await sleep(500)
toast.set(id, { next: 0.1 })

await sleep(3000)
toast.set(id, { next: 0.7 })

await sleep(1000)
toast.set(id, { msg: 'Just a bit more', next: 0.8 })

await sleep(2000)
toast.set(id, { next: 1 })`,
    run: async () => {
      const sleep = t => new Promise(resolve => setTimeout(resolve, t))
      const id = toast.push('Loading, please wait...', { duration: 300, initial: 0, dismissable: false })
      await sleep(500)
      toast.set(id, { next: 0.1 })
      await sleep(3000)
      toast.set(id, { next: 0.7 })
      await sleep(1000)
      toast.set(id, { msg: 'Just a bit more', next: 0.8 })
      await sleep(2000)
      toast.set(id, { next: 1 })
    }
  },
  {
    name: 'CHANGE DEFAULT COLORS',
    code: `<style>
  :root {
    --toastBackground: rgba(255,255,255,0.95);
    --toastColor: #424242;
    --toastProgressBackground: aquamarine;
  }
</style>

<script>
  toast.push('Changed some colors')
<\/script>`, // eslint-disable-line no-useless-escape
    run: () => {
      colors = true
      toast.push('Changed some colors')
    }
  },
  {
    name: 'POSITION TO BOTTOM',
    code: `<style>
  :root {
    --toastContainerTop: auto;
    --toastContainerRight: auto;
    --toastContainerBottom: 8rem;
    --toastContainerLeft: calc(50vw - 8rem);
  }
</style>

<SvelteToast options={{ reversed: true, intro: { y: 192 } }} />

<script>
  toast.push('Bottoms up!')
<\/script>`, // eslint-disable-line no-useless-escape
    run: async () => {
      bottom = true
      options = { reversed: true, intro: { y: 128 } }
      await tick()
      toast.push('Bottoms up!')
    }
  },
  {
    name: 'USE COMPONENT',
    code: `
    toast.push({
        component: {
          src: DummyComponent, 
          props: {
            foo: 'bar'
          }
        }
      })`,
    run: async () => {
      toast.push({
        component: {
          src: DummyComponent,
          props: {
            foo: 'bar'
          }
        }
      })
    }
  },
  {
    name: 'RESTORE DEFAULTS',
    code: '// All default settings restored!',
    run: async () => {
      colors = false
      bottom = false
      options = { reversed: false, intro: { x: 256 } }
      await tick()
      toast.push('All themes reset!')
    }
  },
  {
    name: 'CREATE NEW TOAST CONTAINER',
    code: `<style>
  .wrap {
    --toastContainerTop: 0.5rem;
    --toastContainerRight: 2rem;
    --toastContainerBottom: auto;
    --toastContainerLeft: 2rem;
    --toastWidth: 100%;
    --toastMinHeight: 1.5rem;
    --toastBackground: rgba(59,130,246,0.95);
    --toastMsgPadding: 0.25rem 0.5rem;
    font-size: 0.875rem;
  }
</style>

<div class="wrap">
  <SvelteToast target="new" options={{ initial: 0, intro: { y: -64 } }} />
</div>

<div>
  <SvelteToast options={{ initial: 0, intro: { y: -64 } }} />
</div>

<script>
  // Send toast to "new" container
  toast.push('NEW: Multiple toast container support', { target: 'new' });
  // Send toast to "default" container
  toast.push('Default container toast')
<\/script>`, // eslint-disable-line no-useless-escape
    run: () => {
      toast.push('NEW: Multiple toast container support', { target: 'new' })
      toast.push('Default container toast')
    }
  },
  {
    name: 'REMOVE ALL TOASTS FROM CONTAINER',
    code: `// Remove all toasts from "new" container
toast.pop(i => i.target !== 'new')

// Or remove ALL active toasts from ALL containers
toast.pop(0)`,
    run: () => {
      toast.pop(i => i.target !== 'new')
    }
  },
  {
    name: 'SEND COMPONENT AS A MESSAGE',
    code: `toast.push({
  component: {
    src: DummyComponent, // where \`src\` is a Svelte component
    props: {
      title: 'A Dummy Cookie Component'
    },
    sendIdTo: 'toastId' // send toast id to \`toastId\` prop
  },
  dismissable: false,
  initial: 0,
  theme: {
    '--toastMsgPadding': '0'
  }
})`,
    run: () => {
      toast.push({
        component: { src: DummyComponent, props: { title: 'A Dummy Cookie Component' }, sendIdTo: 'toastId' },
        dismissable: false,
        initial: 0,
        theme: { '--toastMsgPadding': '0' }
      })
    }
  },
  {
    name: 'PAUSE ON MOUSE HOVER',
    code: `toast.push('Hover me!', { pausable: true })`, // eslint-disable-line
    run: () => {
      toast.push('Hover me!', { pausable: true })
    }
  }
]
</script>

<style>
.colors {
  --toastBackground: rgba(255,255,255,0.95);
  --toastColor: #424242;
  --toastProgressBackground: aquamarine;
}
.bottom {
  --toastContainerTop: auto;
  --toastContainerRight: auto;
  --toastContainerBottom: 8rem;
  --toastContainerLeft: calc(50vw - 8rem);
}
.top {
  --toastContainerTop: 0.5rem;
  --toastContainerRight: 2rem;
  --toastContainerBottom: auto;
  --toastContainerLeft: 2rem;
  --toastWidth: 100%;
  --toastMinHeight: 1.5rem;
  --toastBackground: rgba(59,130,246,0.95);
  --toastMsgPadding: 0.25rem 0.5rem;
  font-size: 0.875rem;
}
</style>

<div class="container">
  <div class="w-full h-64 px-2 mt-4 mb-8">
    <Prism classes="w-full h-full bg-gray-700 text-gray-200 font-mono shadow rounded-sm overflow-scroll p-4">
      {code}
    </Prism>
  </div>
  <div class="flex flex-row flex-wrap items-center justify-center">
    {#each buttons as btn}
    <button
      class:selected={selected === btn.name}
      on:click={() => { handleClick(btn) }}
      data-btn={camelCase(btn.name)}>
      {btn.name}
    </button>
    {/each}
  </div>
</div>

<div class="top">
  <SvelteToast options={{ initial: 0, intro: { y: -64 } }} target="new" />
</div>

<div class:colors class:bottom>
  <SvelteToast {options} />
</div>
