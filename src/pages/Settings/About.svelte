<!-- About Sup Page -->
<div class="tab-pane">
  <div class="card">
    <div class="card-body">
      <h5 class="card-title">Pano Hakkında</h5>
      <form>
        <div class="form-group row">
          <label class="col-sm-2 col-form-labeel" for="panoVersion">
            Versiyon:
          </label>
          <div class="col">
            <h5
              aria-describedby="panoVersion"
              class="m-0 text-muted"
              id="panoVersion">
              1.0
            </h5>
          </div>
        </div>
        <div class="form-group row">
          <label class="col-sm-2 col-form-labeel" for="siteKeywords">
            Sürüm:
          </label>
          <div class="col">
            <h5
              aria-describedby="panoRelease"
              class="m-0 text-muted"
              id="panoRelease">
              Beta
            </h5>
            <small>Beta sürüm bilinmeyen hatalar içerebilir.</small>
          </div>
        </div>
        <div class="form-group row mb-0">
          <label class="col-sm-2 col-form-labeel" for="siteKeywords">
            Resmi Website:
          </label>
          <div class="col">
            <a
              aria-describedby="panoWebsite"
              href="https://panocms.com"
              id="panoWebsite"
              target="_blank">
              www.panocms.com
            </a>
          </div>
        </div>
        <div class="form-group row mb-0">
          <label class="col-sm-2 col-form-labeel" for="siteKeywords">
            Discord:
          </label>
          <div class="col">
            <a
              aria-describedby="panoWebsite"
              href="https://panocms.com/discord"
              id="panoWebsite"
              target="_blank">
              Pano Discord
            </a>
          </div>
        </div>
      </form>
    </div>
  </div>

  <div class="card">
    <div class="card-body">
      <h5 class="card-title">Açık Kaynak Lisansları</h5>

      <!-- Bootstrap 4.3 License -->
      <details>
        <summary class="h6 text-primary">Bootstrap 4.3</summary>
        <p>
          Lorem ipsum dolor sit amet consectetur adipisicing elit. Quia quisquam
          assumenda dolor eligendi fugit, architecto ab vero possimus minus
          consequatur delectus aut quam voluptatem debitis ullam ea voluptate
          inventore rem!
        </p>
      </details>

      <!-- Another Software License -->
      <details>
        <summary class="h6 text-primary">Another Software</summary>
        <p>
          Lorem ipsum dolor sit amet consectetur adipisicing elit. Quia quisquam
          assumenda dolor eligendi fugit, architecto ab vero possimus minus
          consequatur delectus aut quam voluptatem debitis ullam ea voluptate
          inventore rem!
        </p>
      </details>
    </div>
  </div>
</div>

<script>
  import { isPageInitialized } from "../../Store";
  import { onMount } from "svelte";

  onMount(() => {
    isPageInitialized.set(true);
  });
</script>
