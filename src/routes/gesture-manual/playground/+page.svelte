<script>
  import GestureUI from "../GestureUI.svelte";
  import { onMount } from "svelte";

  let ballEl;
</script>

<GestureUI enableDrag={true} enableClick={true} enableScroll={false}  enableCursor={true}  usePhysicsTransform={true} {ballEl} />


<style>
  :global(.btn-gesture.hovered){
    /* yellow highlight */
    transform: scale(1.08);
    transition: all 0.2s ease-in-out;
  }

    :global(.btn-gesture.pressed) {
    /* tomato color when clicked */
    transform: scale(1);
    transition: all .01s ease;
  }

    @keyframes pop {
    0% {
      transform: scale(1);
      opacity: 1;
    }
    50% {
      transform: scale(1.8);
      opacity: 0.8;
    }
    100% {
      transform: scale(0);
      opacity: 0;
    }
  }

  .ball.popped {
    animation: pop 0.4s forwards;
  }

  .ball {
  position: absolute;
  cursor: grab;
}
</style>

<span class="btn-gesture hovered " style="display: none"></span>
<span class="btn-gesture pressed " style="display: none"></span>
<span class="ball popped" style="display: none"></span>


<main class = "relative">
  <h1 class = "text-center font-semibold text-6xl mt-10">Press, Click, And Drag Away !</h1>
<div bind:this={ballEl} class="draggable ball btn-gesture absolute text-9xl">
  🏀
</div>


</main>
  
  
