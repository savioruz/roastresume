<script>
  import "$src/app.css";
  import { ModeWatcher } from "mode-watcher";
  import { Button } from "$lib/components/ui/button";
</script>

<ModeWatcher />
<main class="flex flex-col h-screen justify-center items-center">
  <div class="flex flex-col items-center">
    <h1 class="text-[120px] font-extrabold text-gray-700">404</h1>
    <p class="text-2xl font-medium text-gray-600 mb-6">Page Not Found</p>
    <Button href="/" class="mt-4 ease-in-out">Go Home</Button>
  </div>
</main>
