<script>
  import GestureUI from "../GestureUI.svelte";
  import { onMount } from "svelte";

  let ballEl;
  let x = 500, y = 200;
  let vx = 0, vy = 0;
  let isDragging = false;
  let dragStart = { x: 0, y: 0 };
  let gravity = 0.6;
  let friction = 0.98;

  function updatePosition() {
      if (!isDragging) {
          vy += gravity;
          x += vx;
          y += vy;

          // 🧱 Halfway floor (center of screen minus half ball height)
          const floor = window.innerHeight / 2 - 48;

          if (y > floor) {
              y = floor;
              vy *= -0.7; // bounce back up (dampened)
              vx *= 0.9;  // reduce horizontal speed slightly
          }

          // Wall collisions (optional, keep it clean)
          const maxX = window.innerWidth - 96;
          if (x < 0 || x > maxX) {
              vx *= -0.7;
              x = Math.max(0, Math.min(x, maxX));
          }

          vx *= friction;
          vy *= friction;
      }

      if (ballEl) {
          ballEl.style.transform = `translate(${x}px, ${y}px)`;
      }

      requestAnimationFrame(updatePosition);
  }
  
  onMount(() => {
    ballEl = document.querySelector(".draggable");

    ballEl.addEventListener("pointerdown", (e) => {
      isDragging = true;
      dragStart.x = e.clientX;
      dragStart.y = e.clientY;
      e.target.setPointerCapture(e.pointerId);
    });

    ballEl.addEventListener("pointermove", (e) => {
      if (isDragging) {
        x = e.clientX - 48;
        y = e.clientY - 48;
      }
    });

    ballEl.addEventListener("pointerup", (e) => {
      if (isDragging) {
        const dx = e.clientX - dragStart.x;
        const dy = e.clientY - dragStart.y;
        vx = dx * 0.3;
        vy = dy * 0.3;
        isDragging = false;
      }
    });

    updatePosition();
  });

</script>

<GestureUI enableDrag={true} enableClick={true} enableScroll={false}  enableCursor={true}  usePhysicsTransform={true} {ballEl} />

<span class="btn-gesture hovered" style="display: none"></span>
<span class="btn-gesture pressed" style="display: none"></span>

<style>
  .btn-gesture.hovered {
    background-color: #0015ff !important; /* yellow highlight */
    transform: scale(1.08);
    transition: all 0.2s ease-in-out;
  }

  .btn-gesture.pressed {
    background-color: rgb(255, 3, 32) !important; /* tomato color when clicked */
    transform: scale(2);
    transition: all 0.01s ease;
  }


</style>

<main class = "relative">
  <h1 class = "text-center font-semibold text-6xl mt-10">Press, Click, Scroll, And Drag Away !</h1>
<div bind:this={ballEl} class="draggable btn-gesture w-24 h-24 text-9xl rounded-full bg-blue-600 text-white flex items-center justify-center font-bold absolute" style="transform: translate(500px, 200px)">
  🏀
</div>

</main>
  
  
