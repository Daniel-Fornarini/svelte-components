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
  function hasHeader() {}

  // TODO:
  function hasFooter() {}

  function onkeypress() {
    if (visible) {
      hide();
    }
  }
</script>

<div class="modal-wrapper">
  {#if visible}
    <!-- TODO: add class without-header -->
    <div transition:fade class="modal fade show" tabindex="-1" role="dialog" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered modal-{size}" role="document">
        <!-- TODO: add click outside -->
        <div class="modal-content {contentClasses}">
          <div class="modal-header {headerClasses}">
            <!-- TODO: has header -->
            <h5 class="modal-title">
              <slot name="header" />
            </h5>
          </div>
          <div class="modal-body {bodyClasses}">
            <slot />
          </div>
          <!-- TODO: has footer -->
          <div class="modal-footer {footerClasses}">
            <slot name="footer" />
          </div>
        </div>
      </div>
    </div>
    <div transition:fade class="modal-backdrop fade show" on:click|preventDefault={hide} />
  {/if}
</div>

<style lang="scss" scoped>
  .modal {
    overflow-y: auto;
    &.show {
      display: block;
    }
  }
  .fade-enter-active {
    transition: opacity 0.5s;
  }
  .fade-leave-active {
    transition: opacity 0.25s;
  }
  .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    opacity: 0;
  }

  .modal-backdrop {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: rgba(#000, 0.7);
    backdrop-filter: blur(3px);
    z-index: 100;
  }
</style>
