<script>
  export let persist = false
  export let persistKey = "theme"
  export let theme = "white"
  export const themes = ["white", "g10", "g90", "g100"]

  import { onMount, afterUpdate, setContext } from "svelte"
  import { writable, derived } from "svelte/store"


  const isValidTheme = value => themes.includes(value)
  const isDark = value => isValidTheme(value) && (value === "g90" || value === "g100")
  
  const dark = writable(isDark(theme))
  const light = derived(dark, x => !x)

  setContext("Theme", {
    updateVar: (name, value) => {
      document.documentElement.style.setProperty(name, value)
    },
    dark,
    light
  })

  onMount(() => {
    try{
      const persitedTheme = localStorage.getItem(persistKey)
      if (isValidTheme(persitedTheme))
        theme = persitedTheme
    }
    catch (error) {
      console.log(error)
    }
  })

  afterUpdate(() => {
    if (isValidTheme(theme)) {
      document.documentElement.setAttribute("theme", theme)
      if (persist)
        localStorage.setItem(persistKey, theme)
    }
    else {
      console.warm(`"${theme}" is not a valid Carbon theme. Choose from available thems: ${JSON.stringify(themes)}`)
    }
  })

  $: dark.set(isDark(theme))
</script>

<slot></slot>