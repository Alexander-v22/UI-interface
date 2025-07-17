<script>
  import 'prismjs/themes/prism-tomorrow.css';
  import Prism from 'prismjs';
  import 'prismjs/components/prism-javascript';
  import { onMount } from 'svelte';

  const codeSample = `
  function handleRightPressAndHold(hand) {
    const isFist = isFistGesture(hand);
    if (!lastRightPalmPosition) return;

    const rect = canvas.getBoundingClientRect();
    const scaleX = canvas.width / video.videoWidth;
    const scaleY = canvas.height / video.videoHeight;

    const scaledX = lastRightPalmPosition.x * scaleX;
    const scaledY = lastRightPalmPosition.y * scaleY;

    const canvasX = canvas.width - scaledX; // mirrored
    const canvasY = scaledY;

    const screenX = rect.left + (canvasX / canvas.width) * rect.width;
    const screenY = rect.top + (canvasY / canvas.height) * rect.height;

    const element = document.elementFromPoint(screenX, screenY);

    if (isFist) {
        if (!isHolding && element) {
            console.log("Fist detected — Pressing on element:", element);
            isHolding = true;
            holdElement = element;

            if (element.classList.contains('btn-gesture')) {
                element.classList.add('pressed');
            }

            element.dispatchEvent(new MouseEvent("mousedown", {
                bubbles: true,
                cancelable: true,
                composed: true,
                clientX: screenX,
                clientY: screenY,
                view: window
            }));
        }
    } 
    
    if(!isFist){
        // RELEASE when no longer a fist (even if not fully open)
        if (isHolding && holdElement) {
            console.log("Hand uncurled — Releasing element:", holdElement);

            holdElement.dispatchEvent(new MouseEvent("mouseup", {
                bubbles: true,
                cancelable: true,
                composed: true,
                clientX: screenX,
                clientY: screenY,
                view: window
            }));

            if (holdElement.classList.contains('btn-gesture')) {
                holdElement.classList.remove('pressed');
            }

            isHolding = false;
            holdElement = null;
        }
    }
}
        
    
    
    `.trim();

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
  <h2 class="text-3xl font-bold mb-4">Drag Gesture Logic</h2>
  <p class="text-gray-600 mb-4">
    These functions detect drag gesture and triggers the drag components
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
