<!-- Site Settings Sub Page -->
<div class="tab-pane">
  <div class="card">
    <div class="card-body">
      <h5 class="card-title">Website Ayarları</h5>
      <form>
        <div class="form-group row">
          <label class="col-md-4 col-form-labeel" for="siteTitle">
            Site Başlığı:
          </label>
          <div class="col">
            <input
              :value="websiteName"
              aria-describedby="siteTitle"
              class="form-control"
              id="siteTitle"
              type="text" />
          </div>
        </div>
        <div class="form-group row">
          <label class="col-md-4 col-form-labeel" for="siteDesc">
            Site Açıklaması:
          </label>
          <div class="col">
            <textarea
              :value="websiteDescription"
              aria-describedby="siteDesc"
              class="form-control"
              id="siteDesc"
              rows="2"></textarea>
          </div>
        </div>
        <div class="form-group row">
          <label class="col-md-4 col-form-labeel" for="siteKeywords">
            Anahtar Kelimeler:
          </label>
          <div class="col">
            <textarea
              aria-describedby="siteKeywords"
              class="form-control"
              id="siteKeywords"
              rows="2"></textarea>
            <small>
              Anahtar kelimeleri virgül ile ayırın. Örnek: panocraft, minecraft,
              server
            </small>
          </div>
        </div>
        <div class="form-group row">
          <label class="col-md-4 col-form-labeel" for="siteFavicon">
            Favicon:
          </label>
          <div class="col">
            <div class="row">
              <div class="col-auto">
                <img
                  alt="Mevcut Favicon"
                  class="d-block ml-auto"
                  height="48"
                  src="http://icons.iconarchive.com/icons/ampeross/lamond/256/minecraft-icon.png"
                  width="48" />
              </div>
              <div class="col">
                <input class="form-control-file" id="siteFavicon" type="file" />
                <small>
                  Favicon PNG, ICO formatında ve minimum 16x16 boyutlarında
                  olmalıdır.
                </small>
              </div>
            </div>
          </div>
        </div>

        <div style="form-group">
          <button class="btn btn-secondary mt-3" style="submit">Kaydet</button>
        </div>
      </form>
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
