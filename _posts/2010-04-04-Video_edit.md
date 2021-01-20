---
layout: post
title: "Video Edit"
author: "Jeppe Pape"
categories: data science
tags: [sample]
youtubeId: fc_QhHqtcUs
---

<body translate='no' >
<div>
	<video id='local' width='320px' height='180px' controls>
		<source src='/assets/vid/Cyklist.mp4'></source>
	</video>
  <video id='webcam' width='320px' height='180px'></video>
  <video id='output-video' width='320px' height='180px' controls></video>
</div>
<button id='record' disabled>Start Recording</button>
<p id='message'></p>

<script src='https://cpwebassets.codepen.io/assets/common/stopExecutionOnTimeout-157cd5b220a5c80d4ff8e0e70ac069bffd87a61252088146915e8726e5d9f147.js'></script>

<script src='https://unpkg.com/@ffmpeg/ffmpeg@0.9.3/dist/ffmpeg.min.js'></script>


<script>
const { createFFmpeg, fetchFile } = FFmpeg;
const ffmpeg = createFFmpeg({
  log: true });


const webcam = document.getElementById('webcam');
const recordBtn = document.getElementById('record');
const startRecording = () => {
  const rec = new MediaRecorder(webcam.srcObject);
  const chunks = [];

  recordBtn.textContent = 'Stop Recording';
  recordBtn.onclick = () => {
    rec.stop();
    recordBtn.textContent = 'Start Recording';
    recordBtn.onclick = startRecording;
  };

  rec.ondataavailable = e => chunks.push(e.data);
  rec.onstop = async () => {
    transcode(new Uint8Array(await new Blob(chunks).arrayBuffer()));
  };
  rec.start();
};

(async () => {
  webcam.srcObject = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
  await webcam.play();
  recordBtn.disabled = false;
  recordBtn.onclick = startRecording;
})();

const transcode = async webcamData => {
  const message = document.getElementById('message');
  const name = 'record.webm';
  message.innerHTML = 'Loading ffmpeg-core.js';
  await ffmpeg.load();
  message.innerHTML = 'Start transcoding';
  ffmpeg.FS('writeFile', name, await fetchFile(webcamData));
  ffmpeg.FS('writeFile', "record_2.mp4", await fetchFile("/assets/vid/Cyklist.mp4"));  
  //await ffmpeg.run('-i', name, 'output.mp4');
  await ffmpeg.run( '-fflags', '+discardcorrupt', '-i', "record_2.mp4", 'output.mp4');
  //await ffmpeg.run( '-y', '-i', 'record_2.webm', '-i', name, ', -filter_complex "[0:v]scale=1920:1080:force_original_aspect_ratio=1,setsar=1:1,pad=1920:1080:(ow-iw)/2:(oh-ih)/2[0v];[0v][1:v:0]concat=n=2:v=1:a=0[outv]" -map "[outv]" output.mp4');

  message.innerHTML = 'Complete transcoding';
  const data = ffmpeg.FS('readFile', 'output.mp4');
  const video = document.getElementById('output-video');
  video.src = URL.createObjectURL(new Blob([data.buffer], { type: 'video/mp4' }));
};
//# sourceURL=pen.js
    </script>

  

</body>