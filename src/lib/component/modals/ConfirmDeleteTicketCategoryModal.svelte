<!-- Confirm Delete Ticket Category Modal -->
<div
  aria-hidden="true"
  class="modal fade"
  id="{dialogID}"
  role="dialog"
  tabindex="-1">
  <div class="modal-dialog modal-dialog-centered" role="dialog">
    <div class="modal-content">
      <div class="modal-body text-center">
        <div class="pb-3">
          <i class="fas fa-question-circle fa-3x d-block m-auto text-gray"></i>
        </div>
        {#if $category.ticketCount !== 0}
          {$_('components.modals.confirm-delete-ticket-category.title-permanent')}
          <br />
          <br />

          {#each $category.tickets as ticket, index (ticket)}
            <a href="{base}/tickets/ticket/{ticket.id}" target="_blank">
              {ticket.title}
            </a>
            <br />
          {/each}

          {#if $category.ticketCount > 5}
            {$_('components.modals.confirm-delete-ticket-category.more-tickets', {values: {count: $category.ticketCount - 5}})}
          {/if}

          <br />
        {/if}
        {$_('components.modals.confirm-delete-ticket-category.title')}
      </div>
      <div class="modal-footer flex-nowrap">
        <button
          class="btn btn-link col-6 m-0"
          type="button"
          class:disabled="{loading}"
          on:click="{hide}">
          {$_('components.modals.confirm-delete-ticket-category.cancel')}
        </button>
        <button
          class="btn btn-danger col-6 m-0"
          type="button"
          class:disabled="{loading}"
          on:click="{onYesClick}">
          {$_('components.modals.confirm-delete-ticket-category.yes')}
        </button>
      </div>
    </div>
  </div>
</div>

<script context="module">
  import { writable, get } from "svelte/store";

  const dialogID = "confirmDeleteTicketCategory";
  const category = writable({
    tickets: [],
  });

  let callback = (category) => {};
  let hideCallback = (category) => {};
  let modal;

  export function show(newCategory) {
    category.set(newCategory);

    modal = new window.bootstrap.Modal(document.getElementById(dialogID), {
      backdrop: "static",
      keyboard: false,
    });
    modal.show();
  }

  export function setCallback(newCallback) {
    callback = newCallback;
  }

  export function hide() {
    hideCallback(get(category));

    modal.hide();
  }

  export function onHide(newCallback) {
    hideCallback = newCallback;
  }
</script>

<script>
  import { base } from "$app/paths";

  import { showNetworkErrorOnCatch } from "$lib/Store";
  import ApiUtil from "$lib/api.util";

  import {
    show as showToast,
    limitTitle,
  } from "$lib/component/ToastContainer.svelte";
  import TicketCategoryDeletedPermanentlyToast from "$lib/component/toasts/TicketCategoryDeletedPermanentlyToast.svelte";
  import { _ } from "svelte-i18n";

  let loading = false;

  function refreshBrowserPage() {
    location.reload();
  }

  function onYesClick() {
    loading = true;

    showNetworkErrorOnCatch((resolve, reject) => {
      ApiUtil.delete({
        path: `/api/panel/ticket/categories/${get(category).id}`,
      })
        .then((body) => {
          if (body.result === "ok") {
            loading = false;

            hide();

            showToast(TicketCategoryDeletedPermanentlyToast, {
              title: get(category).title,
            });

            callback(get(category));

            resolve();
          } else refreshBrowserPage();
        })
        .catch(() => {
          reject();
        });
    });
  }
</script>
