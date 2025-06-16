<script>
  import { getContext } from "svelte"

  export let customContents
  export let text

  export let variant
  export let size
  export let quiet
  export let disabled

  export let dataSource;
  export let s3DatasourceId;
  export let s3Bucket;

  export let bgColour
  export let bgHover
  export let txtColour
  export let txtHover
  export let brdColour
  export let brdHover


  const { API, styleable, notificationStore } = getContext("sdk");
  const component = getContext("component")

  $: disabled = disabled ? "disabled" : ""

  function downloadFromUrl(url, filename = 'query-result.json') {
    const a = document.createElement('a')
    a.href = url
    a.download = filename
    a.target = '_blank'
    a.rel = 'noopener noreferrer'
    a.style.display = 'none'

    document.body.appendChild(a)
    a.click()
    document.body.removeChild(a)
  }

  async function onClick() {
    console.log("Button clicked", dataSource)
    if (!dataSource || dataSource.type != "query" || !dataSource._id) {
      console.error("No data source provided or missing _id")
      return
    }

    if (!s3DatasourceId || !s3Bucket) {
      console.error("S3 datasource or bucket not configured")
      notificationStore.actions.error("S3 configuration missing")
      return
    }

    try {
      const result = await API.executeQuery({ queryId: dataSource._id})
      console.log("Query result", result.data)
      if (!result || !result.data || !result.data.length) {
        console.error("No data returned from query")
        return
      }
      

      // Convert result to JSON
      const jsonData = JSON.stringify(result.data, null, 2);
      const blob = new Blob([jsonData], { type: 'application/json' });
      const file = new File([blob], 'query-result.json', { type: 'application/json' });

      // Upload to S3
      const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
      const s3Key = `query-results/result-${timestamp}.json`;
      
      const uploadResult = await API.externalUpload({
        datasourceId: s3DatasourceId,
        bucket: s3Bucket,
        key: s3Key,
        data: file,
      });

      console.log("Upload result", uploadResult);
      
      if (uploadResult && uploadResult.url) {
        // Use the returned URL for download
        downloadFromUrl(uploadResult.url, `query-result-${timestamp}.json`);
        notificationStore.actions.success("File downloaded successfully");
      } else {
        throw new Error("No download URL returned from S3 upload");
      }
    } catch (error) {
      console.error("Error executing query or uploading to S3:", error)
      notificationStore.actions.error(`Error: ${error?.message || error}`)
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