<!-- Dashboard Page -->
<div class="container">
  <!-- Welcome Alerts -->
  {#if data.gettingStartedBlocks.welcomeBoard}
  <div
    class="alert alert-success alert-dismissible animate__animated animate__zoomIn mb-3">
    <div class="row">
      <h5 class="mb-3">
        {@html $_("pages.dashboard.welcome-card.description")}
      </h5>
      <div class="col-lg-4 mb-lg-0 mb-3">
        <p>
          {$_("pages.dashboard.welcome-card.connect-server-description")}
        </p>
        <button
          class="btn btn-sm btn-primary"
          data-bs-target="#connectServer"
          data-bs-toggle="modal">
          <i class="fa-solid fa-plus me-2"></i>
          {$_("pages.dashboard.welcome-card.connect-server")}
        </button>
      </div>
      <div class="col-lg-4">
        <ul class="list-unstyled mb-0">
          <li>
            <a class="alert-link" href="{base}/posts/create-post">
              <i class="fa-solid fa-pen me-2"></i>
              {$_("pages.dashboard.welcome-card.publish-your-first-post")}
            </a>
          </li>
          <li>
            <a class="alert-link" href="{base}/view">
              <i class="fa-solid fa-brush me-2"></i>
              {$_("pages.dashboard.welcome-card.change-theme")}
            </a>
          </li>
          <li>
            <a class="alert-link" href="{base}/addons">
              <i class="fa-solid fa-puzzle-piece me-2"></i>
              {$_("pages.dashboard.welcome-card.manage-addons")}
            </a>
          </li>
          <li>
            <a class="alert-link" href="{base}/players">
              <i class="fa-solid fa-user-cog me-2"></i>{$_(
                "pages.dashboard.welcome-card.manage-players",
              )}
            </a>
          </li>
        </ul>
      </div>
      <div class="col-lg-4">
        <ul class="list-unstyled">
          <li>
            <a class="alert-link" href="javascript:void(0);">
              <i class="fa-solid fa-arrow-up-right-from-square me-2"></i>
              {$_("pages.dashboard.welcome-card.themes-and-extensions")}
            </a>
          </li>
          <li>
            <a class="alert-link" href="javascript:void(0);" target="_blank">
              <i class="fa-solid fa-arrow-up-right-from-square me-2"></i>
              {$_("pages.dashboard.welcome-card.documentations")}
            </a>
          </li>
          <li>
            <div class="btn-group pt-3">
              <a
                class="btn btn-sm btn-outline-primary"
                href="https://panomc.com/discord"
                target="_blank">
                <i class="fa-solid fa-globe me-2"></i>
                {$_("pages.dashboard.welcome-card.website")}
              </a>
              <a
                class="btn btn-sm btn-outline-primary"
                href="https://panomc.com/discord"
                target="_blank">
                <i class="fab fa-discord me-2"></i>
                {$_("pages.dashboard.welcome-card.discord")}
              </a>
            </div>
          </li>
        </ul>
      </div>
    </div>

    <button
      type="button"
      title="{$_('pages.dashboard.welcome-card.close-button')}"
      class="btn-close"
      data-bs-dismiss="alert"
      on:click="{onCloseGettingStartedCard}"></button>
  </div>
  {/if}
  <div class="row">
    <div class="col-lg-6">
      <!-- Latest Tickets -->
      {#if hasPermission(Permissions.MANAGE_TICKETS)}
        <div class="card mb-3">
          <div class="card-body">
            <h5 class="card-title">
              {$_("pages.dashboard.last-tickets.title")}
            </h5>

            {#if data.tickets.length === 0}
              <NoContent />
            {:else}
              <div class="table-responsive">
                <table class="table table-hover mb-0">
                  {#each data.tickets as ticket, index (ticket)}
                    <tbody>
                      <tr>
                        <td class="align-middle">
                          <a
                            use:tooltip="{[
                              ticket.writer.username,
                              { placement: 'bottom' },
                            ]}"
                            href="{base}/players/player/{ticket.writer
                              .username}">
                            <img
                              src="https://minotar.net/avatar/{ticket.writer
                                .username}/32"
                              alt="{$_(
                                'pages.dashboard.last-tickets.player-name',
                              )}"
                              class="rounded-circle animate__animated animate__zoomIn"
                              height="32"
                              width="32" />
                          </a>
                        </td>
                        <td class="align-middle text-nowrap">
                          <a
                            href="{base}/tickets/ticket/{ticket.id}"
                            title="{$_('pages.dashboard.last-tickets.view')}"
                            >#{ticket.id} {ticket.title}</a>
                        </td>
                        <td class="align-middle text-nowrap">
                          <TicketStatusBadge status="{ticket.status}" />
                        </td>
                        <td class="align-middle text-nowrap"
                          ><span><Date time="{ticket.lastUpdate}" /></span></td>
                      </tr>
                    </tbody>
                  {/each}
                </table>
              </div>
            {/if}
          </div>
        </div>
      {/if}
    </div>
    <div class="col-lg-6">
      <div class="card mb-3">
        <div class="card-body">
          <h5 class="card-title">Logs</h5>
        </div>
      </div>
    </div>
  </div>
</div>

<script context="module">
  import ApiUtil from "$lib/api.util.js";

  async function loadData({ request }) {
    return new Promise((resolve, reject) => {
      ApiUtil.get({
        path: `/api/panel/dashboard`,
        request,
      }).then((body) => {
        if (body.result === "ok") {
          resolve(body);
        } else {
          reject(body);
        }
      });
    });
  }
  /**
   * @type {import('@sveltejs/kit').PageLoad}
   */
  export async function load(event) {
    const { parent } = event;
    const parentData = await parent();

    let data = {
      gettingStartedBlocks: {
        welcomeBoard: false,
      },
      tickets: [],
    };

    if (parentData.NETWORK_ERROR) {
      return data;
    }

    await loadData({ request: event }).then((body) => {
      data = { ...data, ...body };
    });

    return data;
  }
</script>

<script>
  import { getContext } from "svelte";
  import { _ } from "svelte-i18n";

  import { base } from "$app/paths";

  import { showNetworkErrorOnCatch } from "$lib/Store";
  import { hasPermission, Permissions } from "$lib/auth.util";
  import tooltip from "$lib/tooltip.util";

  import NoContent from "$lib/component/NoContent.svelte";
  import TicketStatusBadge from "$lib/component/badges/TicketStatusBadge.svelte";
  import Date from "$lib/component/Date.svelte";

  export let data;

  const pageTitle = getContext("pageTitle");

  pageTitle.set("pages.dashboard.title");

  function onCloseGettingStartedCard() {
    showNetworkErrorOnCatch((resolve, reject) => {
      ApiUtil.post({
        path: "/api/panel/dashboard/closeGettingStartedCard",
      })
        .then(() => {
          resolve();
        })
        .catch(() => {
          reject();
        });
    });
  }
</script>
