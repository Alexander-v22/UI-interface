<script>
  import '../app.css';
  let gestureDropdown = false;
</script>

<div class="flex min-h-screen">
  <!-- Sidebar -->
  <aside class="w-48 bg-gray-700 text-white p-4 sticky top-0 h-screen flex flex-col space-y-2">
    <a class="py-2 px-3 font-semibold hover:underline" href="/home">Home</a>
    <a class="py-2 px-3 font-semibold hover:underline" href="/distance-ui">Face Distance UI</a>

    <div>
      <button
        class="py-2 px-3 font-semibold hover:underline w-full text-left"
        on:click={() => gestureDropdown = !gestureDropdown}
      >
        Gesture Manual
      </button>

      {#if gestureDropdown}
        <div class="ml-4 mt-1 space-y-1">
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual">Welcome</a>
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual/landmarks">Introduction</a>
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual/cursorAndBtn">Cursor & Button</a>
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual/scroll">Scrolling</a>
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual/drag">Drag</a>
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual/playground">Playground</a>
        </div>
      {/if}
    </div>
  </aside>

  <!-- Main content -->
  <main class="flex-1 overflow-y-auto p-6 h-min-screen">
    <slot />
  </main>
</div>
