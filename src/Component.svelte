<script>
  import { getContext } from "svelte"

  export let customContents
  export let text

  export let variant
  export let size
  export let quiet
  export let disabled

  export let dataSource;

  export let bgColour
  export let bgHover
  export let txtColour
  export let txtHover
  export let brdColour
  export let brdHover


  const { API, styleable } = getContext("sdk");
  const component = getContext("component")

  $: disabled = disabled ? "disabled" : ""

  async function onClick() {
    console.log("Button clicked", dataSource)
    if (!dataSource || dataSource.type != "query" || !dataSource._id) {
      console.error("No data source provided or missing _id")
      return
    }

    try {
      const result = await API.executeQuery({ queryId: dataSource._id})
      console.log("Query result", result.data)
      // Convert result to JSON and trigger download
      const jsonData = JSON.stringify(result.data, null, 2)
      
      const blob = new Blob([jsonData], { type: 'application/json;charset=utf-8' })
      const url = URL.createObjectURL(blob)

      const a = document.createElement('a')
      a.href = url
      a.download = 'query-result.json'
      a.target = '_self'  // to ważne dla CSP
      a.rel = 'noopener noreferrer' // zabezpieczenie
      a.style.display = 'none'
      document.body.appendChild(a)
      a.click()
      document.body.removeChild(a)
      URL.revokeObjectURL(url)
      console.log("Download triggered")
    } catch (error) {
      console.error("Error executing query:", error)
    }
  }

</script>

<div use:styleable={$component.styles} class="downloadbutton">
  <button on:click={onClick} class="
            spectrum-Button spectrum-Button--size{size}
            spectrum-Button--{variant}
            {quiet === true ? ' spectrum-Button--quiet' : ''}
          " {disabled}
          style="
            --bgColour:{bgColour}; --bgHover:{bgHover};
            --txtColour:{txtColour}; --txtHover:{txtHover};
            --brdColour:{brdColour}; --brdHover:{brdHover};
          ">
    {#if customContents}
      <slot />
    {:else}
      {text}
    {/if}
  </button>
</div>

<style>
  .spectrum-Button--custom {
    background-color: var(--bgColour, black);
    color: var(--txtColour, white);
    border-color: var(--brdColour, white);
  }

  .spectrum-Button--custom:hover {
    background-color: var(--bgHover, black);
    color: var(--txtHover, white);
    border-color: var(--brdHover, white);
  }
</style>