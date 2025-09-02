<script>
    import GestureUI from "../GestureUI.svelte";
    import DragCode from "./DragCode.svelte";
  
</script>


<GestureUI enableDrag={true} enableClick={false} enableScroll={false}  enableCursor={false}  usePhysicsTransform={false}
/>

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
    transform: scale(1);
    transition: all .01s ease;
  }
</style>


<main class="flex-1 max-w-6xl mx-auto px-4 sm:px-6 md:px-10 lg:px-20 justify-center">

  <h1 class="text-3xl sm:text-4xl md:text-5xl py-5 text-center">Hand Gesture Drag Demo</h1>

  <p class="text-base sm:text-lg mt-5 text-center md:text-left">
    This demo lets you experiment with gesture-based dragging to interact with UI elements. Below you can find a GIF demonstrating the basics and <strong>further down</strong> you can find instructions as well as the <strong>practice demo</strong>.
  </p>

  <section>
    <img
      src="/video-demos/DragDemo.gif"
      alt="Cursor And Button"
      class="mt-6 sm:mt-8 md:mt-10 rounded-xl shadow-lg w-full max-w-[400px] mx-auto"
    />
  </section>

  <!-- Instructions -->
  <div class="flex flex-col md:flex-row gap-6 px-2 sm:px-4 md:px-10 mt-6">
    <div class="flex-1">
      <h2 class="py-2 text-xl sm:text-2xl mt-2">Using Your Hand to Drag</h2>
      <p class="text-base sm:text-lg px-2 sm:px-4">
        Your <strong>left hand</strong> controls the interaction. A virtual "cursor" is positioned at the center of your palm, with the <strong>palm center</strong> (landmark 9) tracked in real-time to move the ball smoothly across the screen.
      </p>
      <ul class="list-disc px-6 sm:px-10 text-base sm:text-lg mt-2">
        <li>The palm cursor follows the movement of your palm on your left hand.</li>
        <li>Position tracking is refreshed approximately 30 times per second.</li>
        <li>Good lighting and clear visibility improve precision and accuracy.</li>
      </ul>
    </div>

    <div class="flex-1">
      <h2 class="py-2 text-xl sm:text-2xl mt-2">Grabbing with a Gesture</h2>
      <p class="text-base sm:text-lg px-2 sm:px-4">
        To simulate the “grab” feature, simply close your <strong>left hand</strong> into a fist. The system interprets this gesture as a grab, allowing you to pick up and move objects. Opening your hand again releases the grab.
      </p>
      <ul class="list-disc px-6 sm:px-10 text-base sm:text-lg mt-2">
        <li>The app monitors hand landmarks to detect when your hand is in a closed (fist) position.</li>
        <li>When a fist is detected, the object under the palm cursor is "grabbed."</li>
        <li>The object stays attached to the palm cursor until you open your hand to release it.</li>
        <li>A ~900ms cooldown helps avoid rapid or unintended grab/release cycles.</li>
      </ul>
    </div>
  </div>
</main>

<section class="max-w-6xl mx-auto px-4 py-8 relative mb-200">
<h2 class="text-3xl font-bold mb-6 text-center">Gesture-Draggable Elements</h2>

<div class="draggable btn-gesture w-24 h-24 rounded-full bg-blue-600 text-white flex items-center justify-center font-bold absolute left-[500px] top-[200px]">
Drag Me
</div>

</section>



<DragCode/>
