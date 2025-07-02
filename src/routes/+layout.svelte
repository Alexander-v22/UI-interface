<script>
  import '../app.css';
  let { children } = $props();
</script>

<div class="flex h-screen">
  <nav class="flex flex-col bg-gray-300 w-40 p-2">
    <a class="py-2 px-3 font-semibold hover:underline" href="/distance-ui">Face.ui</a>
    <a class="py-2 px-3 font-semibold hover:underline" href="/gesture-ui">Gesture.ui</a>
  </nav>

  <main class="flex-1 overflow-y-auto p-4">
    {@render children()}
  </main>
</div>
