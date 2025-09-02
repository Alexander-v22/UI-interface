<script>
    import * as handPoseDetection from "@tensorflow-models/hand-pose-detection";
    import * as tf from "@tensorflow/tfjs";
    import "@tensorflow/tfjs-backend-webgl";
    import { onMount, tick } from 'svelte';

    const {
        enableScroll = false,
        enableClick = false,
        enableCursor = false,
        enableDrag = false,
        usePhysicsTransform = false,
        ballEl: incomingBallEl = null
    } = $props();

    let video;
    let canvas;
    let ctx;
    let model;
    let predictions = [];
    let handLandmarks = {left: '', right: '',};
    let lastIndexTip = null;
    let lastLeftPalmPosition = null;
    let lastRightPalmPosition = null;
    let stableHands = [];
    let videoPlaying = $state(false);
    let detectorRunning = $state(false);
    let stream = $state('')
    let animationFrameId = $state('');

    let lastDrawTime = $state('0');
    let lastScrollTime = $state('0');

    let mousedown = $state(false);
    let lastClicked = "";

    let isHolding = $state(false);  
    let holdElement = $state('');
    let hoveredElementsByHand = {
        left: null,
        right: null
    };

    let draggingElement = $state('');
    let isDragging =  $state(false);  
    let draggingHand = $state(''); // "left" or "right"

    onMount(async () => {
        await tick(); // wait for DOM to mount

          if (!ballEl) {
            ballEl = document.querySelector(".draggable");
        }

        if (ballEl) {
            ballWidth = ballEl.offsetWidth;
            ballHeight = ballEl.offsetHeight;
        }

        if (usePhysicsTransform) {
            updatePosition();
        }
    });

    async function startCamera() {
        if (videoPlaying) return;
        if (!video) {
                console.error('Video element is not defined');
                return;
            } else {
            console.log('Video element is defined');
            }
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
        ); // model that were running

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
        if (!model || !video || video.readyState !== 4) return;
        predictions = await model.estimateHands(video);

        // Ensures there is no flickering 
        if (predictions.length > 0) {
            stableHands = predictions;
        }

        if (shouldUpdate(Date.now())) {
            drawHands(stableHands, {
                enableDrag,
                enableScroll,
                enableClick,
                enableCursor
            });
        }
        // Only redraw at a stable frame rate
    } 

    function drawHands(hands, options = {}) {

        const {
            enableDrag = false,
            enableScroll = false,
            enableClick = false,
            enableCursor = false
         } = options
        
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

        // Store hands to process gestures after drawing
        const gestureQueue = [];

        for (const hand of hands) {
            if (!hand.keypoints || hand.keypoints.length < 21) continue;

            const wrist = hand.keypoints[0];
            const wristX = wrist.x * scaleX;
            const wristY = wrist.y * scaleY;

            const isCursorHand = hand.handedness === 'Left';
            const isClickHand = hand.handedness === 'Right';

            // Draw label
            const label = isCursorHand ? "(Я)" : "(⅃)";
            ctx.fillStyle = "red";
            ctx.font = "20px Arial";
            ctx.fillText(label, wristX + 10, wristY - 10);

            // Draw keypoints
            for (const keypoint of hand.keypoints) {
                const x = keypoint.x * scaleX;
                const y = keypoint.y * scaleY;
                ctx.beginPath();
                ctx.arc(x, y, 6, 0, 2 * Math.PI);
                ctx.fillStyle = "black";
                ctx.fill();
            }

            // Collect gestures for after-draw handling
            gestureQueue.push({ hand, isCursorHand, isClickHand });
        }

        ctx.restore();
        
        // Now safely run gesture logic after drawing is complete to avoid flicerking
        for (const { hand, isCursorHand, isClickHand } of gestureQueue) {
            const isFist = isFistGesture(hand);
            const isOpen = !isFist;


            if (isCursorHand && pointerRH(hand) && enableCursor) {
                handleNavigationGesture(hand);
            }
            if (isCursorHand && isScrollDownGesture(hand) && enableScroll) {
                tryScrollDown();

            }
            if (isCursorHand && isScrollUpGesture(hand) && enableScroll) {
                tryScrollUp();
            }

            if (isClickHand && palmLeftPointer(hand) && enableDrag) {
                handleLeftNavigationPalmGesture(hand);
                handleLeftPalmPressAndHold(hand);
                handleDragAndDrop(hand, isFist, isOpen, 'left', lastLeftPalmPosition);
            }


            if (isCursorHand && palmRightPointer(hand)){
                handleRightNavigationPalmGesture(hand);
                handleRightPressAndHold(hand);
                handleDragAndDrop(hand, isFist, isOpen, 'right', lastRightPalmPosition);
            }

            if (isClickHand && indexToThumbLH(hand) && !isFistGesture(hand) && enableClick) {
                if (!mousedown) {
                    console.log("mousedown");
                    handleClickGesture();
                    mousedown = true;
                }
            }
            if (isClickHand && indexFromThumbLH(hand) && !isFistGesture(hand) && enableClick) {
                if (mousedown) {
                    console.log("mouseup");
                    mousedown = false;
                }
            }

        }
    }

    function pointerRH(hand) {
        const indexTip = hand.keypoints[8];
        const indexMCP = hand.keypoints[5];
        return indexTip.y < indexMCP.y
    }
    
    function palmLeftPointer(hand){
        const middleFingerMCP = hand.keypoints[9];
        const thumbCMC = hand.keypoints[1];
        return middleFingerMCP.y < thumbCMC.y;
    }

     function palmRightPointer(hand){
        const middleFingerMCP = hand.keypoints[9];
        const thumbCMC = hand.keypoints[1];
        return middleFingerMCP.y < thumbCMC.y;
    }
    
    // right hand pointer or curse helper function
    function indexToThumbLH(hand){
        const indexTip = hand.keypoints[8];
        const thumbTip = hand.keypoints[4];
        
        const dx =indexTip.x - thumbTip.x;
        const dy = indexTip.y - thumbTip.y
        const distance = Math.sqrt(dx * dx + dy *dy)
        // This math is need to create the distance so essentially its creating the rate of change at which the position of the thumb and the index finger changes 

        const threshold = 30
        
        return  distance < threshold;
    }

        // helper function for the okay clicker using the right hand 
    function indexFromThumbLH(hand){
    const indexTip = hand.keypoints[8];
        const indexPIP = hand.keypoints[6];
        const thumbTip = hand.keypoints[4];

        const dx = indexTip.x - thumbTip.x;
        const dy = indexTip.y - thumbTip.y;
        const distance = Math.sqrt(dx * dx + dy * dy);

        // Check if index is relatively straight
        const extended = indexTip.y < indexPIP.y - 10; // pointing outward

        const touching = distance >= 40;
        return touching && extended;
    } 
        

    function isFistGesture(hand) {
        const fingerTips = [8, 12, 16, 20];     // Index to Pinky
        const fingerMCPs = [5, 9, 13, 17];      // Their base joints

        const wrist = hand.keypoints[0];
        const middleMCP = hand.keypoints[9];

        // Use wrist to middle finger MCP as a size reference
        const dxRef = wrist.x - middleMCP.x;
        const dyRef = wrist.y - middleMCP.y;
        const refLength = Math.sqrt(dxRef * dxRef + dyRef * dyRef);

        let curledFingers = 0;

        for (let i = 0; i < fingerTips.length; i++) {
            const tip = hand.keypoints[fingerTips[i]];
            const mcp = hand.keypoints[fingerMCPs[i]];

            const dx = tip.x - mcp.x;
            const dy = tip.y - mcp.y;
            const dz = (tip.z || 0) - (mcp.z || 0);

            const dist = Math.sqrt(dx * dx + dy * dy + dz * dz);

            if (dist < refLength * 0.65) { // Adjust multiplier to taste
                curledFingers++;
            }
        }

        return curledFingers >= 4;
    }

    //isnt being used right now 
    // function isOpenHandGesture(hand) {

    //     // Finger tip and MCP joint indices in order: Thumb, Index, Middle, Ring, Pinky
    //     const fingerTips = [4, 8, 12, 16, 20];
    //     const fingerMCPs = [2, 5, 9, 13, 17];

    //     const wrist = hand.keypoints[0];
    //     const middleMCP = hand.keypoints[9];

    //     const refLength = Math.hypot(
    //         wrist.x - middleMCP.x,
    //         wrist.y - middleMCP.y
    //     ); // baseline hand size

    //     let extendedFingers = 0;

    //     for (let i = 0; i < fingerTips.length; i++) {
    //         const tip = hand.keypoints[fingerTips[i]];
    //         const mcp = hand.keypoints[fingerMCPs[i]];

    //         const dist = Math.hypot(
    //             tip.x - mcp.x,
    //             tip.y - mcp.y,
    //             (tip.z || 0) - (mcp.z || 0)
    //         );

    //         if (dist > refLength * 0.7) {
    //             extendedFingers++;
    //         }
    //     }

    //     //console.log("Open hand");
    //     return extendedFingers === 5;
    // }

    // Index tip below MCP (like a finger flicking down)
    function isScrollDownGesture(hand) {
        
        const indexTip = hand.keypoints[8];
        const indexMCP = hand.keypoints[5];

        return indexTip.y > indexMCP.y + 40; // offset for better trigger
    }

    // Index tip and middle finger tip above MCP (like a finger flicking down)
    function isScrollUpGesture(hand) {
        const indexTip = hand.keypoints[8];
        const indexMCP = hand.keypoints[5];

        const middleTip = hand.keypoints[12];
        const middleMCP = hand.keypoints[9];

        const indexIsUp= indexTip.y < indexMCP.y - 20;
        const middleIsUp= middleTip.y < middleMCP.y - 20; 

        return indexIsUp && middleIsUp ;
    }

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

        const canvasX = indexTip.x;
        const canvasY = indexTip.y;

        const rect = canvas.getBoundingClientRect();
        const mirroredX = canvas.width - canvasX;

        const screenX = rect.left + ((mirroredX / canvas.width) * rect.width);
        const screenY = rect.top + ((canvasY / canvas.height) * rect.height);

        const element = document.elementFromPoint(screenX, screenY);

        // Track hover for this hand (assuming this is left hand)
        const handId = "left";

        if (element && element.classList.contains('btn-gesture')) {
            if (hoveredElementsByHand[handId] !== element) {
                if (hoveredElementsByHand[handId]) {
                    hoveredElementsByHand[handId].classList.remove('hovered');
                }
                element.classList.add('hovered');
                hoveredElementsByHand[handId] = element;
            }
        } else if (hoveredElementsByHand[handId]) {
            hoveredElementsByHand[handId].classList.remove('hovered');
            hoveredElementsByHand[handId] = null;
        }
    }

    function handleClickGesture() {
        if (!lastIndexTip) return;

        const rect = canvas.getBoundingClientRect();
        const scaleX = canvas.width / video.videoWidth;
        const scaleY = canvas.height / video.videoHeight;

        const scaledX = lastIndexTip.x * scaleX;
        const scaledY = lastIndexTip.y * scaleY;

        const canvasX = canvas.width - scaledX; // mirrored
        const canvasY = scaledY;

        const screenX = rect.left + (canvasX / canvas.width) * rect.width;
        const screenY = rect.top + (canvasY / canvas.height) * rect.height;

        const element = document.elementFromPoint(screenX, screenY);

        if (element && element.classList.contains("btn-gesture")) {
            element.classList.add("pressed");

            // Only apply pop + physics reset to the ball
            if (element.classList.contains("ball")) {
                // Store current visual position
                const originalTransform = element.style.transform;
                const rect = element.getBoundingClientRect();
                const parentRect = element.offsetParent.getBoundingClientRect();
                const offsetX = rect.left - parentRect.left;
                const offsetY = rect.top - parentRect.top;

                // Freeze current position
                element.style.transition = "none";
                element.style.transform = "none";
                element.style.left = `${offsetX}px`;
                element.style.top = `${offsetY}px`;
                element.style.position = "absolute";
                element.classList.add("popped");

                // Wait for pop animation to finish
                setTimeout(() => {
                    element.style.display = "none";
                    element.classList.remove("popped");

                    // Reset physics state to top-center near title
                    const centerX = window.innerWidth / 2 - 240;
                    const centerY = 0;
                    x = centerX;
                    y = centerY;
                    vx = 0;
                    vy = 9.8;

                    // Apply new transform that physics loop picks up
                    element.style.left = "";
                    element.style.top = "";
                    element.style.position = "";
                    element.style.transform = `translate(${centerX}px, ${centerY}px)`;

                    // Reset any drag-related state
                    element.setAttribute("data-x", 0);
                    element.setAttribute("data-y", 0);

                    // Bring ball back
                    setTimeout(() => {
                        element.style.display = "flex";
                        element.style.transition = "all 0.3s ease";
                    }, 50);
                }, 400);
            }

            // Cleanup press effect after frame
            requestAnimationFrame(() => {
                requestAnimationFrame(() => {
                    element.classList.remove("pressed");
                });
            });

            // Optional simulated mouseup click
            element.dispatchEvent(new MouseEvent("mouseup", {
                bubbles: true,
                cancelable: true,
                composed: true,
                clientX: screenX,
                clientY: screenY,
                view: window
            }));
        }
    }

    function handleLeftNavigationPalmGesture(hand) {
        const middleFingerMCP = hand.keypoints[9];
        lastLeftPalmPosition = middleFingerMCP;
        lastRightPalmPosition = middleFingerMCP;

        const canvasX = middleFingerMCP.x;
        const canvasY = middleFingerMCP.y;

        const rect = canvas.getBoundingClientRect();
        const mirroredX = canvas.width - canvasX;

        const screenX = rect.left + ((mirroredX / canvas.width) * rect.width);
        const screenY = rect.top + ((canvasY / canvas.height) * rect.height);

        const element = document.elementFromPoint(screenX, screenY);

        // Track hover for this hand (assuming this is right hand)
        const handId = "right";

        if (element && element.classList.contains('btn-gesture')) {
            if (hoveredElementsByHand[handId] !== element) {
                if (hoveredElementsByHand[handId]) {
                    hoveredElementsByHand[handId].classList.remove('hovered');
                }
                element.classList.add('hovered');
                hoveredElementsByHand[handId] = element;
            }
        } else if (hoveredElementsByHand[handId]) {
            hoveredElementsByHand[handId].classList.remove('hovered');
            hoveredElementsByHand[handId] = null;
        }
    }
    
    function handleLeftPalmPressAndHold(hand) {
    const isFist = isFistGesture(hand);
    if (!lastLeftPalmPosition) return;

    const rect = canvas.getBoundingClientRect();
    const scaleX = canvas.width / video.videoWidth;
    const scaleY = canvas.height / video.videoHeight;

    const scaledX = lastLeftPalmPosition.x * scaleX;
    const scaledY = lastLeftPalmPosition.y * scaleY;

    const canvasX = canvas.width - scaledX; // mirrored
    const canvasY = scaledY;

    const screenX = rect.left + (canvasX / canvas.width) * rect.width;
    const screenY = rect.top + (canvasY / canvas.height) * rect.height;

    const element = document.elementFromPoint(screenX, screenY);

    if (isFist) {
        if (!isHolding && element) {
            console.log("Fist detected — Pressing on element with palm:", element);
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

    if (!isFist) {
        if (isHolding && holdElement) {
            console.log("Palm open — Releasing element:", holdElement);

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

     function handleRightNavigationPalmGesture(hand) {
        const middleFingerMCP = hand.keypoints[9];
        lastRightPalmPosition = middleFingerMCP;

        const canvasX = middleFingerMCP.x;
        const canvasY = middleFingerMCP.y;

        const rect = canvas.getBoundingClientRect();
        const mirroredX = canvas.width - canvasX;

        const screenX = rect.left + ((mirroredX / canvas.width) * rect.width);
        const screenY = rect.top + ((canvasY / canvas.height) * rect.height);

        const element = document.elementFromPoint(screenX, screenY);

        // Track hover for this hand (assuming this is right hand)
        const handId = "right";

        if (element && element.classList.contains('btn-gesture')) {
            if (hoveredElementsByHand[handId] !== element) {
                if (hoveredElementsByHand[handId]) {
                    hoveredElementsByHand[handId].classList.remove('hovered');
                }
                element.classList.add('hovered');
                hoveredElementsByHand[handId] = element;
            }
        } else if (hoveredElementsByHand[handId]) {
            hoveredElementsByHand[handId].classList.remove('hovered');
            hoveredElementsByHand[handId] = null;
        }
    }

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
                //console.log("Fist detected — Pressing on element:", element);
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

    let ballEl = incomingBallEl || null;
    let ballWidth = 96;
    let ballHeight = 96;

    let x = 500, y = 200;
    let vx = 0, vy = 0;
    let gravity = 0.6;
    let friction = 0.98;
    let palmHistory = [
        { x: 0, y: 0, t: 0 }, // older
        { x: 0, y: 0, t: 0 }, // newer
    ];

    let lastPalmPosition = { x: 0, y: 0, t: 0 };
    let smoothed = { x: 0, y: 0 }; // persistent smoothed position
    let physicsStarted = false;

  

    // Velocity tracking
   
    function handleDragAndDrop(hand, isFist, isOpen, handId, palmPosition) {
        if (!palmPosition) return;

        const rect = canvas.getBoundingClientRect();
        const scaleX = canvas.width / video.videoWidth;
        const scaleY = canvas.height / video.videoHeight;

        const scaledX = palmPosition.x * scaleX;
        const scaledY = palmPosition.y * scaleY;

        const canvasX = canvas.width - scaledX;
        const canvasY = scaledY;

        const screenX = rect.left + (canvasX / canvas.width) * rect.width;
        const screenY = rect.top + (canvasY / canvas.height) * rect.height;

        const now = performance.now();
        //this is to get high resolution timestamps and than later smoothed out 

        // Start drag
        if (isFist && !isDragging) {

            const element = document.elementFromPoint(screenX, screenY);
            if (element && element.classList.contains("draggable")) {

                draggingElement = element;
                draggingHand = handId;
                isDragging = true;

                element.style.transition = "none";//disable any animation for precisiton 

                palmHistory[0] = { x: screenX, y: screenY, t: now };
                palmHistory[1] = { x: screenX, y: screenY, t: now };
                //needed to calcualte for the physics engine 

                smoothed.x = screenX;
                smoothed.y = screenY;
             }
        }

        // While dragging
        if (isDragging && draggingHand === handId && draggingElement) {
            // Smooth hand position
            const smoothingFactor = 0.5;

            smoothed.x = lerp(smoothed.x, screenX, smoothingFactor);
            smoothed.y = lerp(smoothed.y, screenY, smoothingFactor);

            palmHistory[0] = { ...palmHistory[1] };
            palmHistory[1] = { x: smoothed.x, y: smoothed.y, t: now };

           
            const offsetX = 150;
            const offsetY = 100;

            if (usePhysicsTransform) {
                x = smoothed.x - ballWidth / 2 - offsetX;
                y = smoothed.y - ballHeight / 2 - offsetY;
                
            } else {
            const containerRect = draggingElement.parentElement.getBoundingClientRect();
            // this gets the element that is being dragged DOMS position and setting it equal to the contaier rect .
            const offsetX = smoothed.x - containerRect.left - draggingElement.offsetWidth / 2;
            const offsetY = smoothed.y - containerRect.top - draggingElement.offsetHeight / 2;
            draggingElement.style.left = `${offsetX}px`;
            draggingElement.style.top = `${offsetY}px`;
            }
        }

        // On release
        if (isOpen && isDragging && draggingHand === handId) {
            console.log(`Dropped with ${handId}`);

            const dt = (palmHistory[1].t - palmHistory[0].t) / 1000;
            const dx = palmHistory[1].x - palmHistory[0].x;
            const dy = palmHistory[1].y - palmHistory[0].y;

            if (dt > 0) {
                vx = dx / dt * 0.05;
                vy = dy / dt * 0.05;
                } else {
                vx = 0;
                vy = 0;
            }

            const maxV = 60;
            vx = Math.max(Math.min(vx, maxV), -maxV);
            vy = Math.max(Math.min(vy, maxV), -maxV);

            isDragging = false;
            draggingHand = "";
            draggingElement = null;
        }
    }

    function updatePosition() {
        if (physicsStarted) return;
        physicsStarted = true;

        function loop() {
            if (!isDragging) {
                vy += gravity;
                x += vx;
                y += vy;

                const floor = window.innerHeight/2 -96;
                const roof = -96;

                if (y > floor) {
                    y = floor;
                    vy *= -0.7; // bounce back up (dampened)
                    vx *= 0.9;  // reduce horizontal speed slightly
                }
                if (y < roof){
                    y = roof
                    
                    vy *= -0.7; // bounce back down (dampened)
                    vx *= 0.9;  // reduce horizontal speed slightly

                }

                // Wall collisions (optional, keep it clean)
                const maxX = window.innerWidth - 288;

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
            requestAnimationFrame(loop);
        }

        loop();
    }

    //IMPORTANT this is a math funciton needed to updated theb balls position between old and new (linear interpolation)
    function lerp(a, b, t) {
        return a + (b - a) * t;
    }

    function shouldUpdate(now) {
        //prevent flickering
        const debounce = 1000 / 30; // 30fps
        if (now - lastDrawTime > debounce) {
            lastDrawTime = now;
            return true;
        }
        return false;
    }
</script>

<button
  class="fixed z-50
         top-[200px] left-2                 
         md:left-[15rem] lg:left-[16.5rem]   
         px-1 py-2 rounded-2xl shadow
         text-white bg-gray-700
         text-md
         md:text-base 
         lg:text-md lg:top-4"                          
  onclick={toggleCamera}
>
  {detectorRunning ? 'Stop Camera' : 'Start Camera'}
</button>

<video bind:this={video} autoplay playsinline muted width="10" height="10"  class="fixed top-0 left-0 z-10 bottom-6 right opacity-0  ${detectorRunning ? "opacity-100" : "opacity-0" } "></video>

<canvas width="1280" height="720" class="fixed top-0 left-0 z-50 pointer-events-none w-full h-full ${detectorRunning ? "opacity-100" : "opacity-0"} " bind:this={canvas}></canvas>


<!-- This bottom needs to be inputed into the page directly for it work don't know why it wont work if its imported -->
 
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

  .ball-wrapper {
    position: absolute;
    /* Width/height optional if needed */
  }


  .ball.popped {
    animation: pop 0.4s forwards;
  }
</style>

<span class="btn-gesture hovered " style="display: none"></span>
<span class="btn-gesture pressed " style="display: none"></span>
<span class="ball popped" style="display: none"></span>
<span class="ball-wrapper" style="display: none"></span>