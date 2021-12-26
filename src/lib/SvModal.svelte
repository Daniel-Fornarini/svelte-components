<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { tick } from 'svelte';
  import { fade } from 'svelte/transition';

  export let contentClasses = '';
  export let headerClasses = '';
  export let bodyClasses = '';
  export let footerClasses = '';
  export let hidesOnClickOut = true;
  export let size = 'md';

  export const show = async () => {
    visible = true;
    dispatch('shown');
    await tick();
    hideable = true;
    registerKeyDownEvents();
    console.log($$slots.header);
  };

  const dispatch = createEventDispatcher();

  let visible = false;
  let hideable = false;

  function eventListener(e: KeyboardEvent) {
    if (e.keyCode === 27 && hidesOnClickOut) {
      hide();
    }
  }

  function registerKeyDownEvents() {
    window.addEventListener('keydown', eventListener);
  }

  function unregisterKeyDownEvents() {
    window.removeEventListener('keydown', eventListener);
  }

  function onOutsideClick() {
    if (hideable && this.hidesOnClickOut) {
      hide();
    }
  }

  function hide() {
    visible = false;
    hideable = false;
    dispatch('hidden');
    unregisterKeyDownEvents();
  }

  // TODO:
  function hasHeader() {
    return $$slots.header;
  }

  // TODO:
  function hasFooter() {
    return $$slots.footer;
  }

  function onkeypress() {
    if (visible) {
      hide();
    }
  }
</script>

{#if visible}
  <div class="modal-wrapper" class:active={visible}>
    <div transition:fade class="modal" tabindex="-1" role="dialog" aria-hidden="true">
      <div class="modal-{size}" role="document">
        <div class="modal-content {contentClasses}">

          {#if hasHeader()}
            <div class="modal__header {headerClasses}">
              <h5 class="modal__title">
                <slot name="header" />
              </h5>
            </div>
          {/if}

          <div class="modal__body {bodyClasses}">
            <slot />
          </div>

          {#if hasFooter()}
            <div class="modal__footer {footerClasses}">
              <slot name="footer" />
            </div>  
          {/if}

        </div>
      </div>
    </div>
    <div transition:fade class="modal-backdrop" on:click|preventDefault={hide} />
  </div>
{/if}

<style lang="scss" scoped>
  .modal-wrapper {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    justify-content: center;
    align-items: center;
    z-index: 100;

    &.active {
      display: flex;
    }
  }

  .modal-backdrop {
    position: absolute;
    left: 0;
    right: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(#000, 0.7);
    backdrop-filter: blur(3px);
  }

  .modal {
    z-index: 200;
  }
</style>
