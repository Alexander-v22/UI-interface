<script>
  import { onMount } from 'svelte';
  import * as faceapi from 'face-api.js';

  let videoEl;
  let distance = '...';
  let message = 'How is your day going ???';
  let messageSize = '';
  let videoPlaying = false;
  let detectorRunning = false;
  let stream;

  const getCenter = (points) =>
    points.reduce((acc, p) => ({ x: acc.x + p.x, y: acc.y + p.y }), { x: 0, y: 0 });

  async function startCamera() {
    if (videoPlaying) return;

    await faceapi.nets.tinyFaceDetector.loadFromUri('/models');
    await faceapi.nets.faceLandmark68Net.loadFromUri('/models');

    stream = await navigator.mediaDevices.getUserMedia({
      video: {
        width: { ideal: 1280 },
        height: { ideal: 720 },
        aspectRatio: 16 / 9,
        facingMode: "user"
      }
    });

    videoEl.srcObject = stream;
    await videoEl.play();
    videoPlaying = true;
    detectorRunning = true;

    videoEl.onloadedmetadata = () => detect();
  }

  function stopCamera() {
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
      stream = null;
      videoPlaying = false;
      detectorRunning = false;
    }
  }

  function toggleCamera() {
    if (detectorRunning) {
      stopCamera();
    } else {
      startCamera();
    }
  }

  async function estimateDistance(result) {
    const landmarks = result.landmarks;
    const leftEye = landmarks.getLeftEye();
    const rightEye = landmarks.getRightEye();

    const left = getCenter(leftEye);
    const right = getCenter(rightEye);
    left.x /= leftEye.length;
    left.y /= leftEye.length;
    right.x /= rightEye.length;
    right.y /= rightEye.length;

    const eyeDistance = Math.hypot(right.x - left.x, right.y - left.y);
    const calibratedK = 12000;
    const mm = calibratedK / eyeDistance;
    distance = `${mm.toFixed(1)} mm`;

    if (mm < 150) {
      messageSize = 'text-lg';
    } else if (mm < 400) {
      messageSize = 'text-2xl';
    } else if (mm < 550) {
      messageSize = 'text-4xl';
    } else {
      messageSize = 'text-6xl';
    }
  }

  async function detect() {
    const result = await faceapi
      .detectSingleFace(videoEl, new faceapi.TinyFaceDetectorOptions())
      .withFaceLandmarks();

    if (result) {
      estimateDistance(result);
    }

    if (detectorRunning) {
      requestAnimationFrame(detect);
    }
  }
</script>

<button class="px-2 bg-gray-500 rounded-2xl" on:click={toggleCamera}>
  {detectorRunning ? 'Stop Camera' : 'Start Camera'}
</button>

<div class="flex flex-col items-center justify-center min-h-screen text-center space-y-6">
  <div class="flex justify-center w-full">
    <video
      bind:this={videoEl}
      autoplay
      playsinline
      muted
      width="200"
      height="200"
      class={`rounded-2xl shadow border-2 border-green-400 ${detectorRunning ? "opacity-100" : "opacity-0"}`}
    ></video>
  </div>
  <p class="p-2 text-md bg-green-50 rounded-2xl border-2 border-green-400 font-medium">
    Estimated distance: {distance}
  </p>
  <h1 class={`transition-all duration-300 ease-in-out p-4 bg-green-50 border-2 border-green-400 rounded-2xl ${messageSize}`}>
    {message}
  </h1>
</div>
