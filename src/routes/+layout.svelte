<script>
  import '../app.css';
  let gestureDropdown = false;
</script>

<div class="min-h-screen md:h-[100dvh] md:flex bg-gray-50">
  <header class="md:hidden sticky top-0 z-10 bg-gray-800 text-white">
    <div class="px-4 py-3 text-base font-semibold">Gesture & Face-Aware UI</div>
  </header>

  <nav class="md:hidden bg-gray-700 text-white px-3 py-2 space-y-1">
    <a class="block py-2 px-2 rounded hover:bg-gray-600" href="/home">Home</a>
    <a class="block py-2 px-2 rounded hover:bg-gray-600" href="/distance-ui">Face Distance UI</a>

    <button
      class="w-full text-left py-2 px-2 rounded hover:bg-gray-600"
      on:click={() => (gestureDropdown = !gestureDropdown)}
      aria-expanded={gestureDropdown}
    >
      Gesture Manual
    </button>

    {#if gestureDropdown}
      <div class="ml-3 space-y-1 pb-1">
        <a class="block py-1 px-2 rounded hover:bg-gray-600" href="/gesture-manual">Welcome</a>
        <a class="block py-1 px-2 rounded hover:bg-gray-600" href="/gesture-manual/landmarks">Introduction</a>
        <a class="block py-1 px-2 rounded hover:bg-gray-600" href="/gesture-manual/cursorAndBtn">Cursor & Button</a>
        <a class="block py-1 px-2 rounded hover:bg-gray-600" href="/gesture-manual/scroll">Scrolling</a>
        <a class="block py-1 px-2 rounded hover:bg-gray-600" href="/gesture-manual/drag">Drag</a>
        <a class="block py-1 px-2 rounded hover:bg-gray-600" href="/gesture-manual/playground">Playground</a>
      </div>
    {/if}
  </nav>

  <aside class="hidden md:flex md:w-56 lg:w-60 bg-gray-700 text-white p-4 sticky top-0 h-[100dvh] flex-col space-y-2">
    <a class="py-2 px-3 font-semibold hover:underline" href="/home">Home</a>
    <a class="py-2 px-3 font-semibold hover:underline" href="/distance-ui">Face Distance UI</a>

    <div>
      <button
        class="py-2 px-3 font-semibold hover:underline w-full text-left"
        on:click={() => (gestureDropdown = !gestureDropdown)}
        aria-expanded={gestureDropdown}
      >
        Gesture Manual
      </button>

      {#if gestureDropdown}
        <nav class="ml-4 mt-1 space-y-1">
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual">Welcome</a>
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual/landmarks">Introduction</a>
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual/cursorAndBtn">Cursor & Button</a>
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual/scroll">Scrolling</a>
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual/drag">Drag</a>
          <a class="block py-1 px-2 hover:underline" href="/gesture-manual/playground">Playground</a>
        </nav>
      {/if}
    </div>
  </aside>

  <!-- Main content -->
  <main class="flex-1 p-4 md:p-6 overflow-y-auto">
    <slot />
  </main>
</div>
