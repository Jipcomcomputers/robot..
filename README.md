 <!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Retro Robot Face with Realistic Sound Wave</title>
  <style>
    body {
      background-color: black;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      overflow: hidden; /* Prevent scrollbars in fullscreen mode */
      outline: none; /* Remove outline from body */
    }
    .robot-face {
      position: relative;
      width: 80vw; /* Adjust width as a percentage of viewport width */
      height: 80vw; /* Make height equal to width to keep it square */
      max-width: 800px; /* Limit maximum width */
      max-height: 800px; /* Limit maximum height */
      background-color: black; /* Ensure background color matches body */
    }
    .eye {
      position: absolute;
      top: 15%;
      width: 20%;
      height: 5%;
      background-color: green;
      transition: height 0.2s, transform 0.2s;
    }
    .eye.left {
      left: 10%;
    }
    .eye.right {
      right: 10%;
    }
    .nose {
      position: absolute;
      top: 40%;
      left: 50%;
      width: 10%;
      height: 5%;
      background-color: green;
      transform: translateX(-50%);
    }
    canvas {
      position: absolute;
      bottom: 10%; /* Adjust this value to move the mouth upwards */
      left: 50%;
      width: 80%;
      height: 20%;
      transform: translateX(-50%);
      border: none;
      outline: none; /* Remove outline from canvas */
    }
    .fullscreen-btn {
      position: absolute;
      top: 10px;
      right: 10px;
      background-color: #00ff00;
      border: none;
      padding: 10px 20px;
      color: black;
      cursor: pointer;
      z-index: 1000; /* Ensure button is on top */
    }
    /* Hide button when in fullscreen */
    body.fullscreen .fullscreen-btn {
      display: none;
    }
    @keyframes tvOff {
      0% { opacity: 1; transform: scaleY(1); }
      50% { opacity: 0.5; transform: scaleY(0.1) translateX(-50%); }
      100% { opacity: 0; transform: scaleY(0); }
    }
    @keyframes tvOn {
      0% { opacity: 0; transform: scaleY(0); }
      50% { opacity: 0.5; transform: scaleY(0.1) translateX(-50%); }
      100% { opacity: 1; transform: scaleY(1); }
    }
    .tv-off {
      animation: tvOff 1s forwards;
    }
    .tv-on {
      animation: tvOn 1s forwards;
    }
    @keyframes glitzyEffect {
      0% { filter: brightness(1); }
      50% { filter: brightness(3); }
      100% { filter: brightness(1); }
    }
    .glitzy {
      animation: glitzyEffect 1s infinite;
    }
  </style>
</head>
<body tabindex="0">
  <div class="robot-face" id="robot-face">
    <button class="fullscreen-btn" id="fullscreenBtn">Fullscreen</button>
    <div class="eye left" id="left-eye"></div>
    <div class="eye right" id="right-eye"></div>
    <div class="nose"></div>
    <canvas id="mouthCanvas" width="800" height="200"></canvas>
  </div>

  <script>
    const leftEye = document.getElementById('left-eye');
    const rightEye = document.getElementById('right-eye');
    const canvas = document.getElementById('mouthCanvas');
    const ctx = canvas.getContext('2d');
    const fullscreenBtn = document.getElementById('fullscreenBtn');
    const robotFace = document.getElementById('robot-face');

    let isMuted = false;
    let audioContext;
    let analyser;
    let source;
    let dataArray;

    function setupAudio() {
      if (navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices.getUserMedia({ audio: true })
        .then(stream => {
          audioContext = new (window.AudioContext || window.webkitAudioContext)();
          source = audioContext.createMediaStreamSource(stream);
          analyser = audioContext.createAnalyser();
          analyser.fftSize = 512;
          source.connect(analyser);
          dataArray = new Uint8Array(analyser.frequencyBinCount);
          drawWave();
        })
        .catch(err => console.error('Error accessing audio stream:', err));
      } else {
        console.error('getUserMedia not supported on your browser!');
      }
    }

    function drawWave() {
      if (audioContext) {
        analyser.getByteTimeDomainData(dataArray);
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        const middleY = canvas.height / 2;
        const sliceWidth = canvas.width / dataArray.length;

        ctx.beginPath();
        ctx.moveTo(0, middleY);

        for (let i = 0; i < dataArray.length; i++) {
          const x = i * sliceWidth;
          const y = middleY + (dataArray[i] - 128) * (canvas.height / 256);
          ctx.lineTo(x, y);
        }

        ctx.lineTo(canvas.width, middleY);
        ctx.strokeStyle = '#00ff00';
        ctx.lineWidth = 2;
        ctx.stroke();

        requestAnimationFrame(drawWave);
      }
    }

    function startBlinking() {
      setInterval(() => {
        leftEye.style.height = '2%'; // Make eyes finer when blinking
        rightEye.style.height = '2%'; // Make eyes finer when blinking
        setTimeout(() => {
          leftEye.style.height = '5%';
          rightEye.style.height = '5%';
        }, 200);
      }, 5000);
    }

    function handleKeyDown(event) {
      if (event.key === 'b' || event.key === 'B') {
        leftEye.style.height = '2%'; // Make eyes finer when blinking
        rightEye.style.height = '2%'; // Make eyes finer when blinking
      } else if (event.key === 'a' || event.key === 'A') {
        // Angry eyes
        leftEye.style.transform = 'rotate(20deg)';
        rightEye.style.transform = 'rotate(-20deg)';
        leftEye.style.height = '3%';
        rightEye.style.height = '3%';
      } else if (event.key === 'h' || event.key === 'H') {
        // Happy eyes
        leftEye.style.transform = 'rotate(0deg)';
        rightEye.style.transform = 'rotate(0deg)';
        leftEye.style.height = '5%';
        rightEye.style.height = '5%';
      } else if (event.key === 's' || event.key === 'S') {
        // Sad eyes
        leftEye.style.transform = 'rotate(-20deg)';
        rightEye.style.transform = 'rotate(20deg)';
        leftEye.style.height = '3%';
        rightEye.style.height = '3%';
      } else if (event.key === 'm' || event.key === 'M') {
        if (isMuted) {
          setupAudio();
          isMuted = false;
        } else {
          if (audioContext) {
            audioContext.suspend();
          }
          isMuted = true;
        }
      } else if (event.key === 'f' || event.key === 'F') {
        turnOff();
      } else if (event.key === 'o' || event.key === 'O') {
        turnOn();
      }
    }

    function handleKeyUp(event) {
      if (event.key === 'b' || event.key === 'B') {
        leftEye.style.height = '5%';
        rightEye.style.height = '5%';
      }
    }

    function handleFocus() {
      document.addEventListener('keydown', handleKeyDown);
      document.addEventListener('keyup', handleKeyUp);
    }

    function handleBlur() {
      document.removeEventListener('keydown', handleKeyDown);
      document.removeEventListener('keyup', handleKeyUp);
    }

    function handleFullscreenChange() {
      if (document.fullscreenElement) {
        document.body.classList.add('fullscreen');
        robotFace.style.width = '100vw';
        robotFace.style.height = '100vh';
        robotFace.style.maxWidth = 'none';
        robotFace.style.maxHeight = 'none';
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight / 4;

        // Ensure focus on canvas in fullscreen
        setTimeout(() => {
          canvas.focus();
        }, 100);
      } else {
        document.body.classList.remove('fullscreen');
        robotFace.style.width = '80vw';
        robotFace.style.height = '80vw';
        robotFace.style.maxWidth = '800px';
        robotFace.style.maxHeight = '800px';
        canvas.width = 800;
        canvas.height = 200;
      }
    }

    function turnOff() {
      robotFace.classList.add('glitzy'); // Add glitzy effect before turning off
      setTimeout(() => {
        robotFace.classList.remove('glitzy');
        robotFace.classList.add('tv-off');
        setTimeout(() => {
          robotFace.style.display = 'none';
          if (audioContext) {
            audioContext.suspend();
          }
        }, 1000); // Match the duration of the animation
      }, 500); // Duration of the glitzy effect
    }

    function turnOn() {
      robotFace.style.display = 'block';
      robotFace.classList.add('glitzy'); // Add glitzy effect before turning on
      setTimeout(() => {
        robotFace.classList.remove('glitzy');
        robotFace.classList.remove('tv-off');
        robotFace.classList.add('tv-on');
        setTimeout(() => {
          robotFace.classList.remove('tv-on');
          setupAudio();
        }, 1000); // Match the duration of the animation
      }, 500); // Duration of the glitzy effect
    }

    fullscreenBtn.addEventListener('click', function() {
      if (!document.fullscreenElement) {
        document.documentElement.requestFullscreen();
      } else {
        document.exitFullscreen();
      }
    });

    document.addEventListener('fullscreenchange', handleFullscreenChange);

    window.addEventListener('focus', handleFocus);
    window.addEventListener('blur', handleBlur);

    // Initial focus and event handling setup
    handleFocus();
    startBlinking();
    setupAudio();
  </script>
