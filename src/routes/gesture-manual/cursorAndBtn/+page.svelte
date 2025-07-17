
<script>
    import GestureUI from "../GestureUI.svelte";
    import GestureCodeDisplay from "./GestureCodeDisplay.svelte";
</script>


<GestureUI enableDrag={false} enableClick={true} enableScroll={false}  enableCursor={true} />


<style>
  .btn-gesture.hovered {
    background-color: #ffd700 !important; /* yellow highlight */
    transform: scale(1.08);
    transition: all 0.2s ease-in-out;
  }

  .btn-gesture.pressed {
    background-color: rgb(71, 187, 255) !important; /* tomato color when clicked */
    transform: scale(1);
    transition: all .01s ease;
  }
</style>


<span class="btn-gesture hovered" style="display: none"></span>
<span class="btn-gesture pressed" style="display: none"></span>

<main class="flex-1 max-w-6xl mx-auto px justify-center">

  <h1 class="text-5xl py-5 text-center">Cursor & Button Gesture</h1>

    <p class="text-lg mt-5 ">
        This demo lets you experiment with gesture-based controls to interact with buttons. Below you can find a GIF demonstrating the basics and <strong>further down</strong> you can find instructions as well as the <strong>practice demo</strong>.
    </p>
    <section >
        <img src="/video-demos/CursorANdBtn.gif" alt="Curser And Button" class="mt-5 rounded-xl shadow-lg w-[400px] mx-auto" />
    </section>





    <div class = "flex flex-rows-2 px-10">
        <div>
            <h2 class="py-2 text-2xl mt-4"> Using Your Hand as a Cursor</h2>
            <p class="text-lg px-4">
                Your <strong>right hand</strong> acts as a virtual mouse. The tip of your <strong>index finger</strong> (landmark 8) is tracked in real-time and used as a pointing tool.
            </p>

            <ul class="list-disc px-10 text-lg mt-2">
                <li>The index fingertip moves across the screen like a mouse cursor.</li>
                <li>Tracking works best when your finger is extended and visible to the camera.</li>
                <li>The system updates the fingertip position around 30 times per second.</li>
            </ul>
        </div>

        <div>
            <h2 class="py-2 text-2xl mt-4"> Clicking with a Gesture</h2>
            <p class="text-lg px-4">
                The <strong>left hand</strong> is used to perform a click gesture. A "click" is detected when your <strong>thumb</strong> and <strong>index finger</strong> come close together.
            </p>

            <ul class="list-disc px-10 text-lg mt-2">
                <li>The app measures the distance between the index tip (8) and thumb tip (4).</li>
                <li>If they are within a small threshold (about 30 pixels apart), it triggers a virtual click.</li>
                <li>The click is dispatched to the DOM element located under the <strong>right-hand</strong> cursor.</li>
                <li>A cooldown of ~900ms prevents repeated accidental clicks.</li>
            </ul>
        </div>

    </div>
    
    <section class="max-w-6xlmx-auto px-4 py-8">
            <h2 class="text-3xl font-bold mb-6 text-center"> Gesture-Tested Buttons</h2>
            <div class="grid grid-cols-2 md:grid-cols-3 gap-6 ">

            <button class=" btn-gesture bg-blue-600 text-white p-10 text-4xl rounded-xl">Test Button</button>
            <button class="btn-gesture bg-green-600 text-white p-10 text-4xl rounded-xl">Test Button</button>
            <button class="btn-gesture bg-red-600 text-white p-10 text-4xl rounded-xl">Test Button</button>
    
            </div>
    </section>
  

</main>

<GestureCodeDisplay/>