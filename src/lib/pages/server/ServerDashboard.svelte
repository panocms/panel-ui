<div class="container">
  <div
    class="row justify-content-between mb-3 animate__animated animate__slideInUp">
    <div class="col-4">
      <div
        class="card bg-opacity-25"
        class:bg-success="{data.server.status === ServerStatus.ONLINE}"
        class:bg-danger="{data.server.status === ServerStatus.OFFLINE}">
        <div class="card-body">
          <p
            class="mb-0"
            class:text-success="{data.server.status === ServerStatus.ONLINE}"
            class:text-danger="{data.server.status === ServerStatus.OFFLINE}">
            {$_("pages.server.dashboard.server-status", {
              values: {
                status:
                  data.server.status === ServerStatus.ONLINE
                    ? $_("pages.server.dashboard.online")
                    : $_("pages.server.dashboard.offline"),
              },
            })}
          </p>
        </div>
      </div>
    </div>
    <div class="col-4">
      <div class="card bg-primary bg-opacity-25">
        <div class="card-body">
          <p class="mb-0 text-primary">
            {$_("pages.server.dashboard.player", {
              values: {
                playerCount: data.server.playerCount,
                maxPlayerCount: data.server.maxPlayerCount,
              },
            })}
          </p>
        </div>
      </div>
    </div>
    <div class="col-4">
      <div class="card">
        <div class="card-body">
          <p class="mb-0 text-dark">
            {#if data.server.status === ServerStatus.ONLINE}
              {$_("pages.server.dashboard.player", {
                values: { upTime: getUptime(data.server.startTime, checkTime) },
              })}
            {:else}
              {$_("pages.server.dashboard.last-online")}
              <DateComponent time="{data.server.stopTime}" />
            {/if}
          </p>
        </div>
      </div>
    </div>
  </div>

  <!-- Statistic Table -->
  <div class="card">
    <div class="card-body">
      <h5 class="card-title">{$_("pages.server.dashboard.statistics")}</h5>
      <div class="table-responsive">
        <table class="table table-hover m-0">
          <tbody class="text-muted">
            <tr>
              <th scope="row">{$_("pages.server.dashboard.server-name")}</th>
              <td>{data.server.name}</td>
            </tr>
            <tr>
              <th scope="row">{$_("pages.server.dashboard.server-type")}</th>
              <td>{data.server.type}</td>
            </tr>
            <tr>
              <th scope="row"
                >{$_("pages.server.dashboard.local-ip-address")}</th>
              <td>{data.server.host}:{data.server.port}</td>
            </tr>
            <tr>
              <th scope="row">{$_("pages.server.dashboard.server-version")}</th>
              <td>{data.server.version}</td>
            </tr>
            <tr>
              <th scope="row"
                >{$_("pages.server.dashboard.total-connected-servers")}</th>
              <td>{data.connectedServerCount}</td>
            </tr>
            <tr>
              <th scope="row">{$_("pages.server.dashboard.date-added")}</th>
              <td><DateComponent time="{data.server.acceptedTime}" /></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</div>

<script context="module">
  import ApiUtil from "$lib/api.util.js";

  export const ServerStatus = Object.freeze({
    ONLINE: "ONLINE",
    OFFLINE: "OFFLINE",
  });

  async function loadData({ request, selectedServer }) {
    return new Promise((resolve, reject) => {
      ApiUtil.get({
        path: `/api/panel/servers/${selectedServer.id}/dashboard`,
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
    const { selectedServer } = parentData;

    let data = {
      server: {},
      connectedServerCount: 0,
    };

    if (parentData.NETWORK_ERROR) {
      return data;
    }

    await loadData({ request: event, selectedServer }).then((body) => {
      data = { ...data, ...body };
    });

    return data;
  }
</script>

<script>
  import { getContext, onDestroy, onMount } from "svelte";
  import { differenceInCalendarDays, intervalToDuration } from "date-fns";
  import { _ } from "svelte-i18n";

  import DateComponent from "$lib/component/Date.svelte";

  const pageTitle = getContext("pageTitle");

  pageTitle.set("pages.server.dashboard.title");

  export let data;

  let checkTime = 0;
  let interval;

  function getUptime(time, checkTime) {
    const now = new Date();

    const duration = intervalToDuration({
      start: time,
      end: now,
    });

    const days = differenceInCalendarDays(time, now);
    const hours = duration["hours"];
    const minutes = duration["minutes"];
    const seconds = duration["seconds"];

    return `${days}:${hours}:${minutes}:${seconds}`;
  }

  onMount(() => {
    interval = setInterval(() => {
      checkTime += 1;
    }, 1000);
  });

  onDestroy(() => {
    clearInterval(interval);
  });
</script>
