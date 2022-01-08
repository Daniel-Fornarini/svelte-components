<script context="module" lang="ts">
  export const prerender = true;
</script>

<script lang="ts">
  import SSwitch from '$lib/SvSwitch.svelte';
  import SvModal from '$lib/SvModal.svelte';
  import SvIcon from '$lib/SvIcon.svelte';
  import SvCheckbox from '$lib/SvCheckbox.svelte';
  import SvDatePickerAirBnb from '$lib/SvDatePickerAirBnb.svelte';
  import { format } from 'date-fns';
  import { AirbnbStyleDatepicker } from 'svelte-airbnb-style-datepicker';

  let test: boolean = false;
  let test1: boolean = false;
  let test2: boolean = false;
  let test3: boolean = false;

  let modal;

  let checkbox: boolean = false;

  let date1: string | Date = '';
  let date2: string | Date = '';
</script>

<div class="datepicker-trigger">
  <input
    type="text"
    id="datepicker-trigger"
    placeholder="Select dates"
    value={`${date1} - ${date2}`}
  />
  <!-- <SvDatePickerAirBnb
    dateOne={date1}
    dateTwo={date2}
    on:date-one-selected={(e) => (date1 = e.detail)}
    on:date-two-selected={(e) => (date2 = e.detail)}
    triggerElementId="datepicker-trigger"
  /> -->
  <AirbnbStyleDatepicker
    dateOne={date1}
    dateTwo={date2}
    on:date-one-selected={(e) => (date1 = e.detail)}
    on:date-two-selected={(e) => (date2 = e.detail)}
    triggerElementId="datepicker-trigger"
  />
</div>

<!-- <SvCheckbox
  value={checkbox}
  on:input={(e) => {
    checkbox = e.detail.selected;
  }}>prova</SvCheckbox
>
<div>{checkbox}</div>
<button on:click={() => (checkbox = !checkbox)}>change</button>

<div style="color: red;">
  <SvIcon>tv</SvIcon>
</div>
 -->
<!-- <SSwitch disabled={false} value={test} on:change={() => (test = !test)} /> -->
<!-- <SSwitch disabled={false} value={test1} on:change={() => (test1 = !test1)} />
<SSwitch disabled={false} value={test2} on:change={() => (test2 = !test2)} />
<SSwitch disabled={false} value={test3} on:change={() => (test3 = !test3)} />

<div>{test}</div>

<SvModal bind:this={modal} hidesOnClickOut={true}>
  <h1 slot="header">Lorem ipsum dolor sit amet.</h1>

  <div class="body">
    <p>
      Lorem ipsum dolor sit amet consectetur adipisicing elit. Ratione qui, vero omnis illum fugiat
      molestias alias quaerat totam voluptate beatae.
      <SvIcon>tv</SvIcon>
    </p>
  </div>

  <div slot="footer">
    <div class="footer">
      <button>test</button>
      <button on:click={() => modal.hide()}>test</button>
    </div>
  </div>
</SvModal>

<button on:click={() => modal.show()}>apri</button>

<style>
  .footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    box-sizing: border-box;
    padding: 1rem;
  }

  .body {
    box-sizing: border-box;
    text-align: center;
    padding: 1rem;
  }
</style> -->
