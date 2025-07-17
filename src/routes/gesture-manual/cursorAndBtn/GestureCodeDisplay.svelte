<script>
  import 'prismjs/themes/prism-tomorrow.css';
  import Prism from 'prismjs';
  import 'prismjs/components/prism-javascript';
  import { onMount } from 'svelte';

  const codeSample = `function handleNavigationGesture(hand) {
  const indexTip = hand.keypoints[8];
  lastIndexTip = indexTip;

  const canvasX = indexTip.x;
  const canvasY = indexTip.y;

  const rect = canvas.getBoundingClientRect();
  const mirroredX = canvas.width - canvasX;

  const screenX = rect.left + ((mirroredX / canvas.width) * rect.width);
  const screenY = rect.top + ((canvasY / canvas.height) * rect.height);

  const element = document.elementFromPoint(screenX, screenY);

  document.querySelectorAll('.btn-gesture.hovered').forEach(btn => {
    btn.classList.remove('hovered');
  });

  if (element && element.classList.contains('btn-gesture')) {
    element.classList.add('hovered');
  }
}

function handleClickGesture() {
  if (!lastIndexTip) return;

  const now = Date.now();
  if (now - lastClickTime < 900) return;
  lastClickTime = now;

  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / video.videoWidth;
  const scaleY = canvas.height / video.videoHeight;

  const scaledX = lastIndexTip.x * scaleX;
  const scaledY = lastIndexTip.y * scaleY;

  const canvasX = canvas.width - scaledX;
  const canvasY = scaledY;

  const screenX = rect.left + (canvasX / canvas.width) * rect.width;
  const screenY = rect.top + (canvasY / canvas.height) * rect.height;

  const element = document.elementFromPoint(screenX, screenY);
  console.log("Clicking on:", element);

  if (element) {
    if (element.classList.contains('btn-gesture')) {
      element.classList.add('pressed');
      requestAnimationFrame(() => {
        requestAnimationFrame(() => {
          element.classList.remove('pressed');
        });
      });
    }

    element.dispatchEvent(new MouseEvent("click", {
      bubbles: true,
      cancelable: true,
      composed: true,
      clientX: screenX,
      clientY: screenY,
      view: window
    }));
  }
}`.trim();

  let codeElement;
  let copied = false;

  function copyToClipboard() {
    navigator.clipboard.writeText(codeSample).then(() => {
      copied = true;
      setTimeout(() => copied = false, 1500);
    });
  }

  onMount(() => {
    Prism.highlightElement(codeElement);
  });
</script>

<section class="max-w-5xl mx-auto my-10 px-4">
  <h2 class="text-3xl font-bold mb-4"> Gesture Detection Logic</h2>
  <p class="text-gray-600 mb-4">
    These two functions are responsible for mapping your hand to the screen and performing gesture clicks:
  </p>

  <div class="relative group">
    <button
      onclick={copyToClipboard}
      class="absolute top-3 right-3 bg-white bg-opacity-10 hover:bg-opacity-20 text-sm text-black px-3 py-1 rounded-lg transition-opacity duration-200 opacity-0 group-hover:opacity-100"
    >
      {copied ? 'Copied!' : 'Copy'}
    </button>

    <pre class="rounded-xl overflow-x-auto shadow-lg text-sm">
      <code bind:this={codeElement} class="language-javascript leading-relaxed">
{codeSample}
      </code>
    </pre>
  </div>
</section>
