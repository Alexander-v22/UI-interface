<script>
  import 'prismjs/themes/prism-tomorrow.css';
  import Prism from 'prismjs';
  import 'prismjs/components/prism-javascript';
  import { onMount } from 'svelte';

  const codeSample = `function isScrollDownGesture(hand) {
  // index tip below MCP (like a finger flicking down)
  const indexTip = hand.keypoints[8];
  const indexMCP = hand.keypoints[5];

  return indexTip.y > indexMCP.y + 40; // offset for better trigger
}

// Index finger "Flicks" down to represent scrolling down
function isScrollUpGesture(hand) {
  // index tip and middle finger tip above MCP (like a finger flicking down)
  const indexTip = hand.keypoints[8];
  const indexMCP = hand.keypoints[5];

  const middleTip = hand.keypoints[12];
  const middleMCP = hand.keypoints[9];

  const indexIsUp = indexTip.y < indexMCP.y - 20;
  const middleIsUp = middleTip.y < middleMCP.y - 20;

  return indexIsUp && middleIsUp;
}
// Index and middle finger both flip up to represent scrolling up

function tryScrollDown() {
  const now = Date.now();
  if (now - lastScrollTime > 1000) {
    window.scrollBy({ top: 500, behavior: 'smooth' });
    lastScrollTime = now;
  }
}

function tryScrollUp() {
  const now = Date.now();
  if (now - lastScrollTime > 1000) {
    window.scrollBy({ top: -500, behavior: 'smooth' });
    lastScrollTime = now;
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
  <h2 class="text-3xl font-bold mb-4">Scroll Gesture Logic</h2>
  <p class="text-gray-600 mb-4">
    These functions detect upward/downward gestures and trigger smooth scrolling on the page:
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
