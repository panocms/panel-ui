<!-- Confirm Close Ticket Modal -->
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
        {$selectedTickets.length === 1
          ? $_('components.modals.confirm-delete-ticket.title-single')
          : $_('components.modals.confirm-delete-ticket.title-multi')}
      </div>
      <div class="modal-footer flex-nowrap">
        <button
          class="btn btn-link col-6 m-0"
          type="button"
          class:disabled="{loading}"
          aria-disabled="{loading}"
          on:click="{hide}">
          {$_('components.modals.confirm-delete-ticket.cancel')}
        </button>
        <button
          class="btn btn-danger col-6 m-0"
          type="button"
          class:disabled="{loading}"
          aria-disabled="{loading}"
          on:click="{onYesClick}">
          {$_('components.modals.confirm-delete-ticket.yes')}
        </button>
      </div>
    </div>
  </div>
</div>

<script context="module">
  import { writable, get } from "svelte/store";

  const dialogID = "confirmDeleteTicket";
  const selectedTickets = writable([]);

  let callback = (selectedTickets) => {};
  let hideCallback = (selectedTickets) => {};
  let modal;

  export function show(newSelectedTickets) {
    selectedTickets.set(newSelectedTickets);

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
    hideCallback(get(selectedTickets));

    modal.hide();
  }

  export function onHide(newCallback) {
    hideCallback = newCallback;
  }
</script>

<script>
  import { showNetworkErrorOnCatch } from "$lib/Store";
  import ApiUtil from "$lib/api.util";

  import { show as showToast } from "$lib/component/ToastContainer.svelte";
  import TicketsDeletedPermanentlyToast from "$lib/component/toasts/TicketsDeletedPermanentlyToast.svelte";
  import { _ } from "svelte-i18n";

  let loading;

  function refreshBrowserPage() {
    location.reload();
  }

  function onYesClick() {
    loading = true;

    showNetworkErrorOnCatch((resolve, reject) => {
      ApiUtil.delete({
        path:
          "/api/panel/tickets?ids=" +
          Object.values(get(selectedTickets)).map((id) => parseInt(id)),
      })
        .then((body) => {
          if (body.result === "ok") {
            loading = false;

            hide();

            const count = get(selectedTickets).length;

            showToast(TicketsDeletedPermanentlyToast, { count });

            callback(get(selectedTickets));

            resolve();
          } else refreshBrowserPage();
        })
        .catch(() => {
          reject();
        });
    });
  }
</script>
