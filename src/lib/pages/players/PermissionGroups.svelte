<div class="container">
  <!-- Action Menu -->
  <div
    class="row justify-content-between align-items-center mb-3 animate__animated animate__slideInUp">
    <div class="col-auto">
      <a class="btn btn-link" role="button" href="{base}/players">
        <i class="fas fa-arrow-left me-2"></i>
        Oyuncular
      </a>
    </div>
    <div class="col-auto">
      <a
        href="javascript:void(0);"
        class="btn btn-secondary"
        on:click="{() => onCreatePermissionGroupClick()}">
        <i class="fas fa-plus me-2"></i>
        Yetki Grubu Oluştur
      </a>
    </div>
  </div>

  <div class="card">
    <div class="card-body">
      <!-- Permissions Table -->

      <div class="table-responsive">
        <table class="table table-borderless table-hover mb-0">
          <thead>
            <tr>
              <th class="align-middle text-nowrap" scope="col"></th>
              <th class="align-middle text-nowrap" scope="col">İsim</th>
              <th class="align-middle text-nowrap" scope="col">Etiket </th>
              <th class="align-middle text-nowrap" scope="col">Yetki Adeti</th>
              <th class="align-middle text-nowrap" scope="col">Oyuncu Adeti</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <th scope="row">
                <div class="dropdown position-static">
                  <button
                    type="button"
                    class="btn btn-link btn-sm"
                    aria-expanded="false"
                    aria-haspopup="true"
                    data-bs-toggle="dropdown"
                    href="javascript:void(0);"
                    title="Eylemler">
                    <span class="fas fa-ellipsis-v"></span>
                  </button>
                  <div
                    class="dropdown-menu dropdown-menu-start animate__animated animate__fadeIn">
                    <a class="dropdown-item" href="javascript:void(0);">
                      <i class="fas fa-user-plus me-2"></i>
                      Oyuncu Ekle
                    </a>
                    <a
                      class="dropdown-item link-danger"
                      href="javascript:void(0);">
                      <i class="fas fa-trash me-2"></i>
                      Sil
                    </a>
                  </div>
                </div>
              </th>
              <td class="align-middle text-nowrap">
                <a title="Görüntüle" href="javascript:void(0);">Yetki İsmi</a>
              </td>
              <td class="align-middle text-nowrap">
                <div class="badge bg-light text-black rounded-pill">
                  Etiket Adı
                </div></td>
              <td class="align-middle text-nowrap">1 </td>
              <td class="align-middle text-nowrap">10</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="table-responsive d-none">
        <table class="table table-borderless" style="width: auto !important;">
          <thead>
            <tr class="align-top">
              <!-- Blank Cell -->
              <th scope="col"></th>

              <!-- Perm Group Name & User Pictures Cell -->
              {#each data.permissionGroups as permissionGroup, index (permissionGroup)}
                <th scope="col" class="text-center">
                  <a
                    class="text-capitalize"
                    href="javascript:void(0);"
                    title="Düzenle"
                    on:click="{() =>
                      onShowEditPermissionGroupButtonClick(permissionGroup)}">
                    {permissionGroup.name}
                  </a>
                  <div
                    class="d-flex flex-row flex-row-reverse justify-content-center align-items-center">
                    {#if permissionGroup.userCount > 3}
                      <small>
                        +{permissionGroup.userCount - 3}
                      </small>
                    {/if}
                    {#each permissionGroup.users as user, index (user)}
                      <span
                        class="overlapping-avatar"
                        use:tooltip="{[user, { placement: 'bottom' }]}">
                        <a href="{base}/players/player/{user}">
                          <img
                            class="animate__animated animate__zoomIn"
                            src="https://crafthead.net/avatar/{user}"
                            width="28"
                            height="28"
                            alt="{user}" />
                        </a>
                      </span>
                    {/each}
                  </div>
                </th>
              {/each}
            </tr>
          </thead>
          <tbody>
            {#each data.permissions as permission, index (permission)}
              <tr>
                <th
                  scope="col"
                  use:tooltip="{[
                    'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis vehicula, enim in fermentum accumsan,',
                    { placement: 'right' },
                  ]}">
                  <!--              TODO: Icon system-->
                  <!--              <Icon-->
                  <!--                data="{icon[convertIconName(permission.iconName)]}"-->
                  <!--                class="text-primary d-block m-auto" />-->
                  <small class="mb-0">
                    {permission.name}
                  </small>
                </th>

                <!-- Checkboxes Section -->
                {#each data.permissionGroups as permissionGroup, index (permissionGroup)}
                  <td class="align-middle">
                    <div
                      class="form-check form-switch d-flex justify-content-center align-content-center">
                      <input
                        type="checkbox"
                        class="form-check-input"
                        id="{permission.name}_{permissionGroup.name}"
                        checked="{isPermissionChecked(
                          permissionGroup,
                          permission
                        )}"
                        on:click="{() =>
                          onPermissionClick(permissionGroup, permission)}"
                        disabled="{isPermissionDisabled(
                          permission,
                          permissionGroup,
                          loadingPermissionsList
                        )}" />
                    </div>
                  </td>
                {/each}
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    </div>
  </div>
</div>

<AddEditPermGroupModal />
<ConfirmDeletePermissionGroupModal />

<script context="module">
  import ApiUtil from "$lib/api.util.js";
  import { showNetworkErrorOnCatch } from "$lib/Store.js";

  async function loadData({ request, CSRFToken }) {
    return new Promise((resolve, reject) => {
      ApiUtil.get({
        path: "/api/panel/permissions",
        request,
        CSRFToken,
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
    await parent();

    let data = {
      permissions: [],
      permissionGroups: [],
      permissionGroupPerms: {},
    };

    // if (event.stuff.NETWORK_ERROR) {
    //   output.props.data.NETWORK_ERROR = true;
    //
    //   return output;
    // }

    await loadData({ request: event }).then((body) => {
      data = { ...data, ...body };
    });

    return data;
  }
</script>

<script>
  import { base } from "$app/paths";

  import tooltip from "$lib/tooltip.util.js";
  import { pageTitle, session } from "$lib/Store.js";

  import AddEditPermGroupModal, {
    show as showPermissionGroupAddEditModal,
    setCallback as setCallbackForPermissionGroupAddEditModal,
  } from "$lib/component/modals/AddEditPermGroupModal.svelte";

  import ConfirmDeletePermissionGroupModal, {
    setCallback as setDeletePermissionGroupModalCallback,
    show as showDeletePermissionGroupModal,
  } from "$lib/component/modals/ConfirmDeletePermissionGroupModal.svelte";

  export let data;

  pageTitle.set("Yetkiler");

  if (data.NETWORK_ERROR) {
    showNetworkErrorOnCatch((resolve, reject) => {
      loadData({ CSRFToken: $session.CSRFToken })
        .then((body) => {
          data = { ...data, ...body };

          resolve();
        })
        .catch(() => {
          reject();
        });
    }, true);
  }

  let loadingPermissionsList = [];

  function reloadData() {
    showNetworkErrorOnCatch((resolve, reject) => {
      loadData({ CSRFToken: $session.CSRFToken })
        .then((loadedData) => {
          resolve();

          data = loadedData;
        })
        .catch(() => {
          reject();
        });
    });
  }

  String.prototype.capitalize = function () {
    return this.charAt(0).toUpperCase() + this.slice(1);
  };

  function refreshBrowserPage() {
    location.reload();
  }

  function convertIconName(iconName) {
    const iconNameSplit = iconName.split("-");
    let restOfIconName = "";

    iconNameSplit
      .filter((iconName, index) => index !== 0)
      .forEach((iconName) => {
        restOfIconName += iconName.capitalize();
      });

    return iconNameSplit[0] + restOfIconName;
  }

  function isPermissionChecked(permissionGroup, permission) {
    return (
      permissionGroup.name === "admin" ||
      (typeof data.permissionGroupPerms[permissionGroup.id] === "undefined"
        ? false
        : !!data.permissionGroupPerms[permissionGroup.id].includes(
            permission.id
          ))
    );
  }

  function isPermissionDisabled(
    permission,
    permissionGroup,
    loadingPermissionsList
  ) {
    return (
      permissionGroup.name === "admin" ||
      loadingPermissionsList[permission.name + "_" + permissionGroup.name]
    );
  }

  function onPermissionClick(permissionGroup, permission) {
    loadingPermissionsList[permission.name + "_" + permissionGroup.name] = true;
    const mode = isPermissionChecked(permissionGroup, permission)
      ? "DELETE"
      : "ADD";

    showNetworkErrorOnCatch((resolve, reject) => {
      ApiUtil.put({
        path: `/api/panel/permissionGroups/${permissionGroup.id}/permissions/${permission.id}`,
        body: {
          mode: mode,
        },
        CSRFToken: $session.CSRFToken,
      })
        .then((body) => {
          if (body.result === "ok") {
            loadingPermissionsList[
              permission.name + "_" + permissionGroup.name
            ] = false;

            if (mode === "ADD") {
              if (
                typeof data.permissionGroupPerms[permissionGroup.id] ===
                "undefined"
              )
                data.permissionGroupPerms[permissionGroup.id] = [permission.id];
              else
                data.permissionGroupPerms[permissionGroup.id].push(
                  permission.id
                );
            } else {
              data.permissionGroupPerms[permissionGroup.id].splice(
                data.permissionGroupPerms[permissionGroup.id].indexOf(
                  permission.id
                ),
                1
              );
            }

            resolve();
          } else if (body.error === "NOT_EXISTS") refreshBrowserPage();
          else reject();
        })
        .catch(() => {
          reject();
        });
    });
  }

  function onCreatePermissionGroupClick() {
    showPermissionGroupAddEditModal("create");
  }

  function onShowEditPermissionGroupButtonClick(permissionGroup) {
    showPermissionGroupAddEditModal("edit", permissionGroup);
  }

  function onShowDeletePermissionGroupModalClick(permissionGroup) {
    showDeletePermissionGroupModal(permissionGroup);
  }

  setCallbackForPermissionGroupAddEditModal(() => {
    reloadData();
  });

  setDeletePermissionGroupModalCallback((permissionGroup) => {
    data.permissionGroups.forEach((permGroup, index) => {
      if (permGroup.id === permissionGroup.id) {
        data.permissionGroups.splice(index, 1);
      }
    });

    data.permissionGroups = data.permissionGroups;

    Object.keys(data.permissionGroupPerms).forEach(
      (permissionGroupPermsId, index) => {
        if (permissionGroupPermsId === permissionGroup.id) {
          data.permissionGroups.splice(index, 1);
        }
      }
    );

    data.permissionGroupPerms = data.permissionGroupPerms;
  });
</script>