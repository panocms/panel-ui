<!-- Posts Page -->
<article class="container">
  <!-- Action Menu -->
  <div
    class="row justify-content-between mb-3 animate__animated animate__slideInUp"
  >
    <div class="col-auto">
      <a class="btn btn-link" role="button" href="{base}/posts/categories">
        <i class="fas fa-list-alt mr-1"></i>
        Yazı Kategorileri
      </a>
    </div>
    <div class="col-auto">
      <a
        class="btn btn-secondary ml-auto"
        role="button"
        href="{base}/posts/create-post"
      >
        <i class="fas fa-plus"></i>
        <span class="d-md-inline d-none ml-1">Yazı Oluştur</span>
      </a>
    </div>
  </div>

  <!-- All Posts -->

  <div class="card">
    <div class="card-body">
      <div class="row justify-content-between pb-3 align-items-center">
        <div class="col-md-auto col-12 text-md-left text-center">
          <h5 class="card-title mb-md-0">
            {data.posts_count}
            {data.pageType === PageTypes.PUBLISHED
              ? "Yayınlanmış"
              : data.pageType === PageTypes.DRAFT
              ? "Taslak"
              : data.pageType === PageTypes.TRASH
              ? "Çöp"
              : ""} Yazı
          </h5>
        </div>
        <div class="col-md-auto col-12 text-md-right text-center">
          <div class="btn-group">
            <a
              class:active="{data.pageType === PageTypes.PUBLISHED}"
              class="btn btn-sm btn-outline-light btn-link"
              role="button"
              href="{base}/posts/published"
            >
              Yayınlanmış
            </a>
            <a
              class:active="{data.pageType === PageTypes.DRAFT}"
              class="btn btn-sm btn-outline-light btn-link"
              role="button"
              href="{base}/posts/draft"
            >
              Taslak
            </a>

            <a
              class:active="{data.pageType === PageTypes.TRASH}"
              class="btn btn-sm btn-outline-light btn-link text-danger"
              role="button"
              href="{base}/posts/trash"
            >
              Çöp
            </a>
          </div>
        </div>
      </div>

      <!-- No Posts -->
      {#if data.posts_count === 0}
        <div class="container text-center animate__animated animate__zoomIn">
          <i class="fas fa-sticky-note fa-3x text-dark text-opacity-25 m-3"></i>
          <p class="text-gray">Burada içerik yok.</p>
        </div>
      {:else}
        <!-- Posts Table -->
        <div class="table-responsive animate__animated animate__fadeIn">
          <table class="table mb-0">
            <thead>
              <tr>
                <th scope="col"></th>
                <th class="min-w-200px align-middle" scope="col">Yazı</th>
                <th scope="col align-middle">Kategori</th>
                <th scope="col align-middle">Yazar</th>
                <th scope="col align-middle">Görüntülenme</th>
                <th scope="col align-middle">Son Güncelleme</th>
              </tr>
            </thead>
            <tbody>
              {#each data.posts as post, index (post)}
                <PostRow
                  post="{post}"
                  pageType="{data.pageType}"
                  buttonsLoading="{buttonsLoading}"
                  on:moveToDraft="{(event) => onMoveToDraft(event.detail.id)}"
                  on:publish="{(event) => onPublishClick(event.detail.id)}"
                  on:deletePost="{(event) =>
                    onDeletePostClick(event.detail.post)}"
                />
              {/each}
            </tbody>
          </table>
        </div>
      {/if}
      <!-- Pagination -->
      <Pagination
        page="{data.page}"
        totalPage="{data.total_page}"
        on:firstPageClick="{() => reloadData(1)}"
        on:lastPageClick="{() => reloadData(data.total_page)}"
        on:pageLinkClick="{(event) => reloadData(event.detail.page)}"
      />
      <!-- Pagination End -->
    </div>
  </div>
</article>

<script context="module">
  import ApiUtil from "$lib/api.util.js";
  import { showNetworkErrorOnCatch } from "$lib/store.js";

  import { StatusTypes as PostStatusTypes } from "$lib/pages/PostEditor.svelte";

  export const PageTypes = Object.freeze({
    PUBLISHED: "published",
    DRAFT: "draft",
    TRASH: "trash",
  });

  export const DefaultPageType = PageTypes.PUBLISHED;

  export function getStatusFromPageType(pageType) {
    return pageType === PageTypes.PUBLISHED
      ? PostStatusTypes.PUBLISHED
      : pageType === PageTypes.DRAFT
      ? PostStatusTypes.DRAFT
      : PostStatusTypes.TRASH;
  }

  async function loadData({ page, pageType, request, CSRFToken }) {
    return new Promise((resolve, reject) => {
      ApiUtil.post({
        path: "/api/panel/initPage/postPage",
        body: {
          page: parseInt(page),
          page_type: getStatusFromPageType(pageType),
        },
        request,
        CSRFToken,
      }).then((body) => {
        if (body.result === "ok") {
          const data = body;

          data.page = parseInt(page);
          data.pageType = pageType;

          resolve(data);
        } else {
          reject(body);
        }
      });
    });
  }

  /**
   * @type {import('@sveltejs/kit').Load}
   */
  export async function load(request, pageType = DefaultPageType) {
    let output = {
      props: {
        data: {
          posts_count: 0,
          posts: [],
          total_page: 1,
          page: 1,
        },
      },
    };

    if (request.stuff.NETWORK_ERROR) {
      output.props.data.NETWORK_ERROR = true;

      return output;
    }

    await loadData({ page: request.page.params.page || 1, pageType, request })
      .then((data) => {
        output.props.data = { ...output.props.data, ...data };
      })
      .catch((body) => {
        if (body.error === "PAGE_NOT_FOUND") output = null;
      });

    return output;
  }
</script>

<script>
  import { goto } from "$app/navigation";
  import { page, session } from "$app/stores";
  import { base } from "$app/paths";

  import { pageTitle } from "$lib/store.js";

  import Pagination from "$lib/component/Pagination.svelte";

  import {
    setCallback as setDeletePostModalCallback,
    show as showDeletePostModal,
    onHide as onDeletePostModalHide,
  } from "$lib/component/modals/ConfirmDeletePostModal.svelte";
  import PostRow from "$lib/component/PostRow.svelte";

  export let data;

  pageTitle.set(
    (data.pageType === PageTypes.PUBLISHED
      ? "Yayınlanmış" + " "
      : data.pageType === PageTypes.DRAFT
      ? "Taslak" + " "
      : data.pageType === PageTypes.TRASH
      ? "Çöp" + " "
      : "") + "Yazılar"
  );

  if (data.NETWORK_ERROR) {
    showNetworkErrorOnCatch((resolve, reject) => {
      loadData({
        page: $page.params.page || 1,
        pageType: data.pageType,
        CSRFToken: $session.CSRFToken,
      })
        .then((loadedData) => {
          data = loadedData;

          resolve();
        })
        .catch((body) => {
          if (body.error === "PAGE_NOT_FOUND") {
            goto(base + "/error-404");

            resolve();
          } else {
            reject();
          }
        });
    }, true);
  }

  let buttonsLoading = false;

  function refreshBrowserPage() {
    location.reload();
  }

  function onMoveToDraft(id) {
    buttonsLoading = true;

    showNetworkErrorOnCatch((resolve, reject) => {
      ApiUtil.post({
        path: "/api/panel/post/moveDraft",
        body: { id },
        CSRFToken: $session.CSRFToken,
      })
        .then((body) => {
          if (body.result === "ok") {
            buttonsLoading = false;

            reloadData();

            resolve();
          } else refreshBrowserPage();
        })
        .catch(() => {
          reject();
        });
    });
  }

  function onPublishClick(id) {
    buttonsLoading = true;

    showNetworkErrorOnCatch((resolve, reject) => {
      ApiUtil.post({
        path: "/api/panel/post/onlyPublish",
        body: { id },
        CSRFToken: $session.CSRFToken,
      })
        .then((body) => {
          if (body.result === "ok") {
            buttonsLoading = false;

            goto(base + "/posts");

            resolve();
          } else refreshBrowserPage();
        })
        .catch(() => {
          reject();
        });
    });
  }

  function reloadData(page = data.page, pageType = data.pageType) {
    showNetworkErrorOnCatch((resolve, reject) => {
      loadData({ page, pageType, CSRFToken: $session.CSRFToken })
        .then((loadedData) => {
          resolve();

          if (page !== data.page) {
            goto(base + "/posts/" + data.pageType + "/" + page);
          } else {
            data = loadedData;
          }
        })
        .catch((body) => {
          if (body.error === "PAGE_NOT_FOUND") {
            resolve();

            reloadData(page - 1);
          } else {
            reject();
          }
        });
    });
  }

  function onDeletePostClick(post) {
    data.posts[data.posts.indexOf(post)].selected = true;

    showDeletePostModal(post);
  }

  setDeletePostModalCallback((post) => {
    if (data.posts.indexOf(post) !== -1)
      data.posts[data.posts.indexOf(post)].selected = false;

    reloadData();
  });

  onDeletePostModalHide((post) => {
    if (data.posts.indexOf(post) !== -1)
      data.posts[data.posts.indexOf(post)].selected = false;
  });
</script>