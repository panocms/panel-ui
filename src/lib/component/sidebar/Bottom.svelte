<nav class="nav justify-content-around w-100 py-1 position-absolute bottom-0">
  <a
    class="nav-link text-light"
    href="{base}/help"
    use:tooltip="{[$_('components.bottom.help'), { placement: 'top' }]}">
    <i class="fas fa-question-circle"></i>
  </a>
  <a
    class="nav-link text-light"
    href="javascript:void(0);"
    use:tooltip="{[$_('components.bottom.pano-market'), { placement: 'top' }]}">
    <i class="fas fa-store-alt"></i>
  </a>
  <a
    class="nav-link text-light"
    href="https://discord.panomc.com"
    target="_blank"
    use:tooltip="{[$_('components.bottom.discord'), { placement: 'top' }]}">
    <i class="fab fa-discord"></i>
  </a>
</nav>

<script>
  import { _ } from "svelte-i18n";
  import { base } from "$app/paths";

  import tooltip from "$lib/tooltip.util";
</script>
