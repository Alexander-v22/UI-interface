<script>
  import { onMount } from 'svelte';
  import * as faceapi from 'face-api.js';

  let videoEl;
  let distance = '...';
  let message = 'How is your day going ???';
  let messageSize = '';

  // Utility to get center of eye landmarks
  const getCenter = (points) =>
    points.reduce((acc, p) => ({ x: acc.x + p.x, y: acc.y + p.y }), { x: 0, y: 0 });


  onMount(async () => {
    await faceapi.nets.tinyFaceDetector.loadFromUri('/models');
    await faceapi.nets.faceLandmark68Net.loadFromUri('/models');

    const stream = await navigator.mediaDevices.getUserMedia({ video: {} });
    videoEl.srcObject = stream;

    videoEl.onloadedmetadata = () => {
      detect();
    };
  });
  
  async function estimateDistance(result) {
    const landmarks = result.landmarks;
    const leftEye = landmarks.getLeftEye();
    const rightEye = landmarks.getRightEye();

    // Get eye centers
    const left = getCenter(leftEye);
    const right = getCenter(rightEye);
    left.x /= leftEye.length;
    left.y /= leftEye.length;
    right.x /= rightEye.length;
    right.y /= rightEye.length;

    const eyeDistance = Math.hypot(right.x - left.x, right.y - left.y);

    // Calibrated constant (adjust to match your setup)
    const calibratedK = 12000;
    const mm = calibratedK / eyeDistance;
    distance = `${mm.toFixed(1)} mm`;

    // Message & size based on distance
    if (mm < 150) {
      message;
      messageSize = 'text-md';
    } else if (mm < 200) {
      message ;
      messageSize = 'text-lg';
    } else if (mm < 250) {
      message;
      messageSize = 'text-2xl';
    } else {
      message;
      messageSize = 'text-4xl';
    }

    console.log('Eye distance:', eyeDistance, 'Est. mm:', mm);
  }

  async function detect() {
    const result = await faceapi
      .detectSingleFace(videoEl, new faceapi.TinyFaceDetectorOptions())
      .withFaceLandmarks();

    if (result) {
      estimateDistance(result);
    }

    requestAnimationFrame(detect);
  }
</script>

<div class="flex flex-col items-center justify-center min-h-screen text-center space-y-6 ">
  <div class="flex justify-center w-full">
    <video
      bind:this={videoEl}
      autoplay
      muted
      playsinline
      width="480"
      height="360"
      class="rounded-2xl shadow border-2 border-green-400"
    ></video>
  </div>
  <p class="p-2 text-md bg-green-50 rounded-2xl border-2 border-green-400 font-medium">Estimated distance: {distance}</p>
  <h1 class={`transiton-all duration-300 ease-in-out p-4 bg-green-50 border-2 border-green-400 rounded-2xl  ${messageSize}`}>{message}</h1>
</div>
