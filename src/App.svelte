<script>
  import { Router } from "@roxi/routify";
  import { routes } from "../.routify/routes";
</script>

<style  global>
  @import "../assets/global.css";
  @import "carbon-components-svelte/css/all";
</style>

<Router {routes} />