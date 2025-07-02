<script>
    import * as handPoseDetection from "@tensorflow-models/hand-pose-detection";
    import * as tf from "@tensorflow/tfjs";
    import "@tensorflow/tfjs-backend-webgl";
    import { onMount } from 'svelte';
    import axios from 'axios';

    let video;
    let canvas;
    let ctx;
    let model;
    let predictions = [];
    let handLandmarks = {left: '', right: '',};
    let lastIndexTip = null;
    let stableHands = [];
    let lastDrawTime = $state('0');
    let lastClickTime = $state('0');
    let lastScrollTime = $state('0');


    let videoPlaying = $state(false);
    let detectorRunning = $state(false);
    let stream = $state('')

    let animationFrameId = $state('');



    async function startCamera() {
        if (videoPlaying) return;

        stream = await navigator.mediaDevices.getUserMedia({
            video: {
                width: { ideal: 1280 },
                height: { ideal: 720 },
                aspectRatio: 16 / 9,
                facingMode: "user"
            }
        });

        video.srcObject = stream;

        await video.play();
        videoPlaying = true;

        ctx = canvas.getContext("2d");
        canvas.style.backgroundColor = 'transparent';

        await tf.setBackend("webgl");
        await tf.ready();

        model = await handPoseDetection.createDetector(
            handPoseDetection.SupportedModels.MediaPipeHands,
            {
                runtime: "mediapipe",
                modelType: "lite",
                solutionPath: "https://cdn.jsdelivr.net/npm/@mediapipe/hands",
            }
        );

        detectorRunning = true;
        drawLoop(); // start the hand detection loop
    }

    function stopCamera() {
        if (stream) {
            stream.getTracks().forEach(track => track.stop());
            stream = null;
            videoPlaying = false;
        }

        if (ctx) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        }
    
        if (animationFrameId) {
        cancelAnimationFrame(animationFrameId);
        animationFrameId = null;
        }

        predictions = [];
        stableHands = [];
        detectorRunning = false;
    }


    function toggleCamera() {
        if (detectorRunning) {
            stopCamera();
        } else {
            startCamera();
        }
    }

    async function drawLoop() {
    animationFrameId = requestAnimationFrame(drawLoop);
        //calls its self at the browesrs frame rate 
        if (!model || !video || video.readyState !== 4) return;
        //make sure everything is loaded

        predictions = await model.estimateHands(video);

        if (predictions.length > 0) {
            stableHands = predictions;
        }
        //stable for no flickering logic


        if (shouldUpdate(Date.now())) {
            drawHands(stableHands);
        }
        // Only redraw at a stable frame rate
    } 

    function drawHands(hands) {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        ctx.save();
        ctx.strokeStyle = "red";
        ctx.lineWidth = 4;
        ctx.strokeRect(0, 0, canvas.width, canvas.height);
        ctx.restore();

        ctx.save();
        ctx.translate(canvas.width, 0);
        ctx.scale(-1, 1);
        const scaleX = canvas.width / video.videoWidth;
        const scaleY = canvas.height / video.videoHeight;

        for (const hand of hands) {
                if (!hand.keypoints || hand.keypoints.length < 21) continue;

                const wrist = hand.keypoints[0];
                const wristX = wrist.x * scaleX;
                const wristY = wrist.y * scaleY;

                const isVisuallyLeft = wrist.x < video.videoWidth / 2;
                const isCursorHand = isVisuallyLeft;
                const isClickHand = !isVisuallyLeft;

                // Draw hand label (scaled)
                const label = isCursorHand ? "(Я)" : "(⅃)";
                ctx.fillStyle = "red";
                ctx.font = "20px Arial";
                ctx.fillText(label, wristX + 10, wristY - 10);

                //  Draw keypoints (scaled)
                for (const keypoint of hand.keypoints) {
                    const x = keypoint.x * scaleX;
                    const y = keypoint.y * scaleY;
                    ctx.beginPath();
                    ctx.arc(x, y, 6, 0, 2 * Math.PI);
                    ctx.fillStyle = "black";
                    ctx.fill();
                }

                //  Handle gestures (use original keypoints in video space)

                //this controls the ability to use ur hand as a cursor and
                // confirmer
                
                // if (isClickHand && indexToThumbLH(hand)) {
                //     handleClickGesture();
                // }
                // if (isCursorHand && pointerRH(hand)) {
                //     handleNavigationGesture(hand);
                // }
                
                if (isCursorHand && isScrollDownGesture(hand)) {
                    tryScrollDown();
                }
                if (isCursorHand && isScrollUpGesture(hand)) {
                    tryScrollUp();
                }
        }
        ctx.restore();
    }

    function pointerRH(hand) {
        const indexTip = hand.keypoints[8];
        const indexMCP = hand.keypoints[5];
        return indexTip.y < indexMCP.y
    }
    // Left hand pointer or curse helper function

    function indexToThumbLH(hand){
        const indexTip = hand.keypoints[8];
        const thumbTip = hand.keypoints[4];
        
        const dx =indexTip.x - thumbTip.x;
        const dy = indexTip.y - thumbTip.y
        const distance = Math.sqrt(dx * dx + dy *dy)
        // This math is need to create the distance so essentially its creating the rate of change at which the position of the thumb and the index finger changes 

        const threshold = 30
        
        return  distance < threshold;
        // this basically 
    }
    // helper function for the okay clicker using the right hand 

    function isScrollDownGesture(hand) {
        // index tip below MCP (like a finger flicking down)
        const indexTip = hand.keypoints[8];
        const indexMCP = hand.keypoints[5];

        return indexTip.y > indexMCP.y + 40; // offset for better trigger
    }

    //index finger "Flicks" down to represent scrolling down
    function isScrollUpGesture(hand) {
        // index tip and middle finger tip above MCP (like a finger flicking down)
        const indexTip = hand.keypoints[8];
        const indexMCP = hand.keypoints[5];

        const middleTip = hand.keypoints[12];
        const middleMCP = hand.keypoints[9];

    // const ringTip = hand.keypoints[16];
        //const ringMCP = hand.keypoints[13];

        //const pinkyTip = hand.keypoints[20];
        //const pinkyMCP = hand.keypoints[17];


        const indexIsUp= indexTip.y < indexMCP.y - 20;
        const middleIsUp= middleTip.y < middleMCP.y - 20; 

        //const ringIsDown= ringTip.y > indexMCP.y + 40;
        //const pinkyDown= middleTip.y > middleMCP.y + 40; 

            return indexIsUp && middleIsUp ;

        
    }
    //Index and middle finger both flip up to represent scrolling up

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
    }
    //The next two bottom funcions are used to control the speed at which the user scrolls

    function handleNavigationGesture(hand) {
        const indexTip = hand.keypoints[8];
        lastIndexTip = indexTip;
        //saves the current index tip for future use 

        const canvasX = indexTip.x;
        const canvasY = indexTip.y;
        //assings the canvas dimensions to the index finger to use like a cursor

        const rect = canvas.getBoundingClientRect();
        //gets the canvas dimensions
        const mirroredX = canvas.width - canvasX;


        const screenX = rect.left + ((mirroredX / canvas.width) * rect.width) ;
        const screenY = rect.top + ((canvasY / canvas.height) * rect.height) ;
        //Converts canvas space to DOM screen space.
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

        const canvasX = canvas.width - scaledX; // mirrored
        const canvasY = scaledY;

        // Convert canvas space to page-relative screen space
        const screenX = window.scrollX + rect.left + (canvasX / canvas.width) * rect.width;
        const screenY = window.scrollY + rect.top + (canvasY / canvas.height) * rect.height;

        const element = document.elementFromPoint(screenX - window.scrollX, screenY - window.scrollY);
        console.log("Clicking on:", element);

        if (element) {
            element.dispatchEvent(new MouseEvent("click", {
                bubbles: true,
                cancelable: true,
                composed: true,
                clientX: screenX,
                clientY: screenY,
                view: window
            }));
        }
    }

    //prevent flickering
    function shouldUpdate(now) {
        const debounce = 1000 / 30; // 30fps
        if (now - lastDrawTime > debounce) {
            lastDrawTime = now;
            return true;
        }
        return false;
    }
</script>

<button class = "px-2 bg-gray-500 rounded-2xl" onclick={toggleCamera}>
  {detectorRunning ? 'Stop Camera' : 'Start Camera'}
</button>

<video bind:this={video} autoplay playsinline muted width="10" height="10"  class="fixed top-0 left-0 z-10 bottom-6 right opacity-0  ${detectorRunning ? "opacity-100" : "opacity-0" } ">
</video>

<canvas width="1280" height="720" class="fixed top-0 left-0 z-50 pointer-events-none w-full h-full ${detectorRunning ? "opacity-100" : "opacity-0"} " bind:this={canvas}>
</canvas>


