<script>
  import '../app.css';
    import GestureUi from './gesture-manual/GestureUI.svelte';
  let { children } = $props();
  let gestureDropdown= $state(false);

</script>

<div class="flex min-h-screen">
  <nav class="flex flex-col bg-gray-300 w-40 p-1">
    <a class="py-2 px-3 font-semibold hover:underline fixed top-2" href="/home">Home</a>
    <a class="py-2 px-3 font-semibold hover:underline fixed top-12" href="/distance-ui">Face.ui</a>
    

    <div class = "relative">
      <button class="py-2 px-3 font-semibold hover:underline fixed top-22" 
      onclick={() => gestureDropdown =!gestureDropdown} >Gesture Manual</button>

        {#if gestureDropdown}
          <div class = "flex flex-col pl-3 ">
              <a class="py-2 px-3 font-semibold hover:underline fixed top-30" href="/gesture-manual">Welcome</a>
              <a class="py-2 px-3 font-semibold hover:underline fixed top-38" href="/gesture-manual/landmarks">Introduction</a>
              <a class="py-2 px-3 font-semibold hover:underline fixed top-46" href="/gesture-manual/cursorAndBtn">Cursor & button</a>
              <a class="py-2 px-3 font-semibold hover:underline fixed top-54" href="/gesture-manual/scroll">Scrolling</a>
              <a class="py-2 px-3 font-semibold hover:underline fixed top-62" href="/gesture-manual/drag">Drag</a>
              <a class="py-2 px-3 font-semibold hover:underline fixed top-70" href="/gesture-manual/playground">Playground</a>

          </div>
        {/if}

    </div>
  </nav>

  <main class="flex-1 overflow-y-auto p-4">
    <slot/> 
  </main>


</div>