<style global>
  .answer > p {
    margin-bottom: 0;
  }
</style>

<article class="container">
  <div
    class="row justify-content-between mb-3 animate__animated animate__slideInUp">
    <div class="col-auto">
      <a class="btn btn-link" role="button" href="{base}/tickets">
        <i class="fas fa-arrow-left me-2"></i>
        {$_("pages.ticket-detail.tickets")}
      </a>
    </div>
    <div class="col-auto ml-auto">
      <button
        class="btn btn-link link-danger"
        type="button"
        on:click="{() => showDeleteTicketModal([data.ticket.id])}">
        <i class="fas fa-trash"></i>
      </button>
      {#if data.ticket.status !== TicketStatuses.CLOSED}
        <button
          class="btn btn-danger"
          type="button"
          on:click="{() => showCloseTicketModal([data.ticket.id])}">
          <i class="fas fa-times me-2"></i>
          {$_("pages.ticket-detail.close-ticket")}
        </button>
      {/if}
    </div>
  </div>

  <div class="card mb-3">
    <div
      class="card-header py-3 rounded-top"
      class:bg-success="{data.ticket.status === TicketStatuses.NEW}"
      class:bg-warning="{data.ticket.status === TicketStatuses.REPLIED}"
      class:bg-danger="{data.ticket.status === TicketStatuses.CLOSED}">
      <div class="row">
        <div class="col">
          <h5 class="card-title">{data.ticket.title}</h5>
          <small class="mb-0">
            {@html $_("pages.ticket-detail.by-who", {
              values: {
                username: `<a href="${base}/players/player/${data.ticket.username}"
              >${data.ticket.username}</a>`,
              },
            })}
            <Date time="{data.ticket.date}" />,
            {@html $_("pages.ticket-detail.opened-in-category", {
              values: {
                category: `<a href="${base}/tickets/category/${data.ticket.category.url}"
              >${
                data.ticket.category.title === "-"
                  ? $_("pages.ticket-detail.no-category")
                  : data.ticket.category.title
              }</a>`,
              },
            })}
          </small>
        </div>
        <div class="col-auto">
          <TicketStatusBadge status="{data.ticket.status}" />
        </div>
      </div>
    </div>
    <div
      class="card-body messages-section"
      id="messageSection"
      bind:this="{messagesSectionDiv}"
      bind:clientHeight="{$messagesSectionClientHeight}">
      {#if data.ticket.messages.length < data.ticket.count && data.ticket.count > 5}
        <button
          class="btn btn-link bg-light d-block m-auto"
          class:disabled="{loadMoreLoading}"
          on:click="{loadMore}"
          ><i class="fas fa-arrow-up me-2"></i>
          {$_("pages.ticket-detail.previous-messages", {
            values: {
              count:
                data.ticket.count -
                (data.ticket.messages.length - sentMessageCount),
            },
          })}
        </button>
      {/if}

      {#each data.ticket.messages as message, index (message)}
        {#if message.panel}
          <div class="row py-2 flex-nowrap justify-content-end">
            <div class="col d-flex justify-content-end align-items-center">
              <div class="card text-bg-primary">
                <div class="card-header small">
                  <Date time="{message.date}" />
                </div>
                <div class="card-body answer">
                  {@html message.message}
                </div>
              </div>
            </div>
            <div class="col-auto">
              <a href="{base}/players/player/{message.username}">
                <img
                  src="https://minotar.net/avatar/{message.username}/48"
                  alt="{message.username}"
                  class="rounded animate__animated animate__zoomIn"
                  use:tooltip="{[message.username, { placement: 'bottom' }]}"
                  width="48"
                  height="48" />
              </a>
            </div>
          </div>
        {:else}
          <div class="row py-2 flex-nowrap justify-content-start">
            <div class="col-auto">
              <a href="{base}/players/player/{message.username}">
                <img
                  src="https://minotar.net/avatar/{message.username}/48"
                  alt="{message.username}"
                  class="rounded animate__animated animate__zoomIn"
                  use:tooltip="{[message.username, { placement: 'bottom' }]}"
                  width="48"
                  height="48" />
              </a>
            </div>
            <div class="col d-flex flex-nowrap align-items-center">
              <div class="card text-bg-light">
                <div class="card-header small">
                  <Date time="{message.date}" />
                </div>
                <div class="card-body">
                  {message.message}
                </div>
              </div>
            </div>
          </div>
        {/if}
      {/each}

      <!-- Send Message Section -->
      <div
        class="row align-items-end mt-3"
        class:d-none="{data.ticket.status === TicketStatuses.CLOSED}">
        <div class="col d-flex flex-column">
          <!-- Editor -->
          <Editor bind:content="{messageText}" bind:isEmpty="{isEditorEmpty}" />
          <!-- Editor End -->
        </div>
        <div class="col-auto">
          <button
            class="btn btn-secondary mt-lg-0 mt-3"
            on:click="{sendMessage}"
            class:disabled="{messageSendLoading || isEditorEmpty}"
            :disabled="{messageSendLoading || isEditorEmpty}">
            <i class="fas fa-paper-plane"></i>
            <span class="d-lg-inline d-none ms-2"
              >{$_("pages.ticket-detail.send-button")}</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</article>

<script context="module">
  import { writable } from "svelte/store";

  import ApiUtil from "$lib/api.util.js";
  import { showNetworkErrorOnCatch } from "$lib/Store.js";

  import { TicketStatuses } from "$lib/component/badges/TicketStatusBadge.svelte";
  import Editor from "$lib/component/Editor.svelte";
  import { error } from "@sveltejs/kit";

  async function loadTicket({ id, request }) {
    return new Promise((resolve, reject) => {
      ApiUtil.get({
        path: `/api/panel/tickets/${id}`,
        request,
      }).then((body) => {
        if (body.result === "ok") {
          const ticket = body.ticket;

          ticket.id = parseInt(id);

          resolve(ticket);
        } else {
          reject(body);
        }
      });
    });
  }

  // async function loadCategories() {
  //   return new Promise((resolve, reject) => {
  //     ApiUtil.get("panel/post/category/categories")
  //       .then((response) => {
  //         if (response.data.result === "ok") {
  //           const data = response.data;
  //
  //           resolve(data);
  //         } else if (response.data.result === "error") {
  //           const errorCode = response.data.error;
  //
  //           reject(errorCode, response.data);
  //         }
  //       })
  //       .catch((e) => {
  //         console.log(e);
  //       });
  //   });
  // }

  /**
   * @type {import('@sveltejs/kit').PageLoad}
   */
  export async function load(event) {
    const { parent } = event;
    const parentData = await parent();

    let data = {
      ticket: {
        id: -1,
        title: "",
        category: "-",
        username: "",
        status: TicketStatuses.NEW,
        count: 0,
        messages: [],
        date: 0,
      },
    };

    if (parentData.NETWORK_ERROR) {
      return data;
    }

    await loadTicket({ id: event.params.id, request: event })
      .then((body) => {
        data.ticket = body;
      })
      .catch((body) => {
        if (body.error) {
          if (body.error === "NOT_EXISTS" || body.error === "PAGE_NOT_FOUND") {
            throw error(404, body.error);
          }

          throw error(500, body.error);
        }
      });

    return data;
  }
</script>

<script>
  import { afterUpdate, getContext, onMount } from "svelte";
  import { _ } from "svelte-i18n";

  import { goto } from "$app/navigation";
  import { base } from "$app/paths";

  import tooltip from "$lib/tooltip.util.js";

  import {
    setCallback as setCloseTicketModalCallback,
    show as showCloseTicketModal,
  } from "$lib/component/modals/ConfirmCloseTicketModal.svelte";
  import {
    setCallback as setDeleteTicketModalCallback,
    show as showDeleteTicketModal,
  } from "$lib/component/modals/ConfirmDeleteTicketModal.svelte";

  import Date from "$lib/component/Date.svelte";
  import TicketStatusBadge from "$lib/component/badges/TicketStatusBadge.svelte";

  export let data;

  const pageTitle = getContext("pageTitle");

  pageTitle.set("#" + data.ticket.id + " " + limitTitle(data.ticket.title));

  let messagesSectionDiv;
  let loadMoreLoading = false;
  let messageSendLoading = false;

  let messageText = "";
  let isEditorEmpty = true;

  let shouldScroll = true;

  let sentMessageCount = 0;

  const messagesSectionClientHeight = writable(0);

  function loadMore() {
    loadMoreLoading = true;

    showNetworkErrorOnCatch((resolve, reject) => {
      ApiUtil.get({
        path: `/api/panel/tickets/${data.ticket.id}/messages?lastMessageId=${data.ticket.messages[0].id}`,
      })
        .then((body) => {
          if (body.result === "ok") {
            body.messages.reverse().forEach((message) => {
              data.ticket.messages.unshift(message);
            });

            data.ticket.messages = data.ticket.messages;

            loadMoreLoading = false;
          } else if (body.error === "NOT_EXISTS") {
            goto(base + "/error-404");
          } else reject();
        })
        .catch(() => {
          reject();
        });
    });
  }

  function sendMessage() {
    messageSendLoading = true;

    showNetworkErrorOnCatch((resolve, reject) => {
      ApiUtil.post({
        path: `/api/panel/tickets/${data.ticket.id}/message`,
        body: {
          message: messageText,
        },
      })
        .then((body) => {
          if (body.result === "ok") {
            shouldScroll = true;

            data.ticket.messages.push(body.message);

            sentMessageCount++;

            data.ticket.status = TicketStatuses.REPLIED;
            messageText = "";

            messageSendLoading = false;

            resolve();
          } else if (body.error === "NOT_EXISTS") {
            goto(base + "/error-404");
          } else reject();
        })
        .catch(() => {
          reject();
        });
    });
  }

  function limitTitle(text) {
    const limit = 32;

    if (text.length > limit) {
      text = text.substring(0, limit) + "...";
    }

    return text;
  }

  setCloseTicketModalCallback(() => {
    data.ticket.status = TicketStatuses.CLOSED;
  });

  setDeleteTicketModalCallback(() => {
    goto(base + "/tickets");
  });

  afterUpdate(() => {
    if (shouldScroll && messagesSectionDiv.scrollHeight > 0) {
      messagesSectionDiv.scrollTo(0, messagesSectionDiv.scrollHeight);

      shouldScroll = false;
    }
  });

  onMount(() => {
    shouldScroll = true;
  });
</script>
