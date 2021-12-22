<script context="module" lang="ts">
  export const prerender = true;
</script>

<script lang="ts">
  import SSwitch from '$lib/SvSwitch.svelte';
  import SvModal from '$lib/SvModal.svelte';

  let test: boolean = false;

  let modal;
</script>

<SSwitch disabled={false} value={test} on:change={() => (test = !test)} />

<div>{test}</div>

<SvModal bind:this={modal}>
  <span slot="header"> header test</span>

  <div>Test</div>

  <div slot="footer">footer test</div>
</SvModal>

<button on:click={() => modal.show()}>apri</button>
