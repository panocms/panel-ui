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
        {$selectedTickets.length === 1 ? $_('components.modals.confirm-close-ticket.title-single'): $_('components.modals.confirm-close-ticket.title-multi')}
      </div>
      <div class="modal-footer flex-nowrap">
        <button
          class="btn btn-link col-6 m-0"
          data-bs-dismiss="modal"
          type="button"
          class:disabled="{loading}"
          aria-disabled="{loading}"
          on:click="{hide}">
          {$_('components.modals.confirm-close-ticket.cancel')}
        </button>
        <button
          class="btn btn-danger col-6 m-0"
          type="button"
          class:disabled="{loading}"
          aria-disabled="{loading}"
          on:click="{onYesClick}">
          {$_('components.modals.confirm-close-ticket.yes')}
        </button>
      </div>
    </div>
  </div>
</div>

<script context="module">
  import { writable, get } from "svelte/store";

  const dialogID = "confirmCloseTicket";
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
  import { _ } from "svelte-i18n";

  import { showNetworkErrorOnCatch } from "$lib/Store";
  import ApiUtil from "$lib/api.util";

  import { show as showToast } from "$lib/component/ToastContainer.svelte";
  import TicketsClosedToast from "$lib/component/toasts/TicketsClosedToast.svelte";
  import { TicketStatuses } from "$lib/component/badges/TicketStatusBadge.svelte";

  let loading;

  function refreshBrowserPage() {
    location.reload();
  }

  function onYesClick() {
    loading = true;

    showNetworkErrorOnCatch((resolve, reject) => {
      ApiUtil.put({
        path: "/api/panel/tickets",
        body: {
          tickets: Object.values(
            get(selectedTickets).map((id) => parseInt(id))
          ),
          status: TicketStatuses.CLOSED,
        },
      })
        .then((body) => {
          if (body.result === "ok") {
            loading = false;

            hide();

            const count = get(selectedTickets).length;

            showToast(TicketsClosedToast, { count });

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
