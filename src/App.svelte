<script>

import { onMount } from 'svelte';
import { writable } from 'svelte/store';
import { fade } from 'svelte/transition';
import { Prohibition, createDOMRenderer } from 'prohibition'

let door;
let guard;
let recorder;
let renderer;
let recordingRenderer;
let knocking = "";
let state = "scanning";
let secretKnock = [ 2452, 2889, 3177, 3346, 3782, 4697, 5163 ];

let recording = writable(false);

onMount(() => {
  initialize();
})

recording.subscribe(() => {
  state = handleState(state, $recording ? 'recording' : 'recording-cancel')
  if ($recording) {
    guard.stop();
    recorder.start();
  }
})

window.getState = function () {
  console.log(state)
}

function handleState(state, evt) {
  console.log(state, evt)
  let states = {
    scanning: {
      events: {
        knock: 'waiting',
        recording: 'recording'
      }
    },
    waiting: {
      events: {
        success: 'accepted',
        failure: 'rejected'
      }
    },
    rejected: {
      events: {
        wait: 'scanning',
        'recording': 'recording'
      }
    },
    recording: {
      events: {
        'recording-cancel': 'scanning',
        'recording-finish': 'scanning',
      }
    },
    accepted: {
      events: {
        'recording': 'recording'
      }
    }
  };
  if (evt in states[state].events) {
    return states[state].events[evt];
  }
  return state;
}

function getDownEvent() {
  return 'ontouchstart' in window ? 'touchstart' : 'mousedown'
}

function getUpEvent() {
  return 'ontouchend' in window ? 'touchend' : 'mouseup'
}

function handleKnockStart(e) {
  e.preventDefault();
  if (['scanning', 'waiting'].includes(state)) {
    knocking = 'knocking';
  }
}

function handleKnockEnd(e) {
  e.preventDefault();
  if (['scanning', 'waiting'].includes(state)) {
    knocking = '';
  }
}

function initialize() {
  renderer = createDOMRenderer(document.getElementById('recorded-knock'))
  renderer.drawContainer();
  renderer.updateKnock(secretKnock)
  recordingRenderer = createDOMRenderer(document.getElementById('recording-knock'))
  recordingRenderer.drawContainer();

  door = document.querySelector('#door')
  guard = Prohibition.getGuard(door)
  recorder = Prohibition.getRecorder(door)

  guard.listen('knock', function() {
    state = handleState(state, 'knock')
  })

  guard.listen('guardDone', function() {
    door.addEventListener('animationend', () => {
      state = handleState(state, 'wait')
      guard.start()
    })
    let evt = ''
    if (guard.test(secretKnock)) {
      evt = 'success'
    } else {
      evt = 'failure'
    }
    state = handleState(state, evt)
  });
  guard.start()

  recorder.listen('recordDone', function() {
    renderer.updateKnock(recorder.knock)
    recordingRenderer.updateKnock([])
    $recording = false;
  });

  recorder.listen('knockChanged', function(newKnock) {
    recordingRenderer.updateKnock(newKnock)
  })

  // setup "knock" feedback outside of prohibition
  door.addEventListener(getDownEvent(), handleKnockStart)
  door.addEventListener(getUpEvent(), handleKnockEnd)
}

</script>

<main>
<div class="{state}">
  <div class="wrapper">
    <h1>Prohibition</h1>
    <p class="subhead">Secret Knock Authentication</p>
  </div>
  <div id="door-container">
    <div class="wrapper">
      <div id="mask">
        <div id="room"></div>
        <div id="door" class="door-open {knocking}">
          <div id="knob"></div>
          <div id="slat-container">
          <div class="eye left">
              <div class="ball"></div>
            </div>
            <div class="brow left"></div>
            <div class="eye right">
              <div class="ball"></div>
            </div>
            <div class="brow right"></div>
            <div id="slat" class="checking" />
          </div>
        </div>
        <span class="speech scram">Scram!</span><span class="speech ok">Come in.</span>
      </div>
    </div>
  </div>
  <div class="wrapper">
    <label class="toggle">
      <input class="toggle-checkbox" type="checkbox" bind:checked={$recording}>
      <div class="toggle-switch"></div>
      <span class="toggle-label">rec</span>
    </label>
    <div id="visualization-wrapper">
      <div id="recorded-knock"></div>
    </div>
    <div id="recording-visualization-wrapper">
      <div id="recording-knock"></div>
      <p class="caption">knock on the door to record your own secret knock</p>
    </div>
    <h2>What is this?</h2>
    <p>Prohibition is a javascript library providing "secret knock" authentication.</p>
    <p>It allows you to 'record' secret knocks (taps and clicks) and then test knocks against the recording.</p>
    <p>You can supply a callback that is told if knock succeeded or not.</p>
    <p>You can use it wire up a fun&mdash;but very insecure&mdash;authentication system.</p>

    <h2>Who is this for?</h2>
    <p>Me and maybe someone creating a website for a speakeasy? Might be good for hiding easter eggs.</p>

    <h2>How do I use it?</h2>
    <p>If you really want to, download and import the codes.</p>
    <p>A secret knock is just an array of timestamps (more on this below).</p>
      <pre class="hljs" style="display: block; overflow-x: auto; padding: 0.5em; background: rgb(43, 43, 43) none repeat scroll 0% 0%; color: rgb(186, 186, 186);"><span class="hljs-keyword" style="color: rgb(203, 120, 50);">let</span> <span class="hljs-attr">secretKnock</span> = [ <span class="hljs-number" style="color: rgb(104, 150, 186);">2452</span>, <span class="hljs-number" style="color: rgb(104, 150, 186);">2889</span>, <span class="hljs-number" style="color: rgb(104, 150, 186);">3177</span>, <span class="hljs-number" style="color: rgb(104, 150, 186);">3346</span>, <span class="hljs-number" style="color: rgb(104, 150, 186);">3782</span>, <span class="hljs-number" style="color: rgb(104, 150, 186);">4697</span>, <span class="hljs-number" style="color: rgb(104, 150, 186);">5163</span> ];</pre>
    <p>Make a "Guard" by passing a DOM element to <span class="code">getGuard</span> and start it.</p>
    <pre class="hljs" style="display: block; overflow-x: auto; padding: 0.5em; background: rgb(43, 43, 43) none repeat scroll 0% 0%; color: rgb(186, 186, 186);"><span class="hljs-keyword" style="color: rgb(203, 120, 50);">let</span> <span class="hljs-attr">door</span> = document.querySelector('<span class="hljs-comment" style="color: rgb(127, 127, 127);">#door')</span>
<span class="hljs-keyword" style="color: rgb(203, 120, 50);">let</span> <span class="hljs-attr">guard</span> = Prohibition.getGuard(
guard.start()</pre>
    <p>This element will now listen to knocks. If enough time passes, a <span class="code">guardDone</span> event will fire. Listen to it and perform a test:</p>
    <pre class="hljs" style="display: block; overflow-x: auto; padding: 0.5em; background: rgb(43, 43, 43) none repeat scroll 0% 0%; color: rgb(186, 186, 186);">door.listen(<span class="hljs-string" style="color: rgb(224, 196, 108);">'guardDone'</span>, <span class="hljs-function"><span class="hljs-params" style="color: rgb(185, 185, 185);">()</span> =&gt;</span> <span class="hljs-built_in" style="color: rgb(224, 196, 108);">console</span>.log(guard.test(secretKnock)) }</pre>
    <p>The test function will return true if the secretKnock is close to the knock your user has just performed.</p>
    <p>In order to make a secret knock, you can create a recorder. It's almost exactly the same as a Guard:</p>
    <pre class="hljs" style="display: block; overflow-x: auto; padding: 0.5em; background: rgb(43, 43, 43) none repeat scroll 0% 0%; color: rgb(186, 186, 186);">let door = <span class="hljs-built_in" style="color: rgb(224, 196, 108);">document</span>.querySelector(<span class="hljs-string" style="color: rgb(224, 196, 108);">'#door'</span>)
let recorder = Prohibition.getRecorder(door)
recorder.listen(<span class="hljs-string" style="color: rgb(224, 196, 108);">'recordDone'</span>, <span class="hljs-function"><span class="hljs-params" style="color: rgb(185, 185, 185);">(secretKnock)</span> =&gt;</span> <span class="hljs-built_in" style="color: rgb(224, 196, 108);">console</span>.log(secretKnock) }
recorder.start()</pre>
    <p>You can paste those values right into an array or store them programatically.</p>
    <p>There are a few other events you can listen to: <span class="code">knock</span>, <span class="code">knockChanged</span>.</p>
    <h2>What else?</h2>
    <p>The "testing" function is very simple. It normalizes all the values and checks how close they are against the reference. Feel free to write your own machine learning powered detector.</p>
  </div>
</div>
</main>

<style>

.code {
  font-family: monospace;
}

#door-container {
  background: black;
}

.speech {
  position: absolute;
  background: white;
  color: black;
  padding: 10px;
  border-radius: 3px;
  border: solid black 1px;
  top: 175px;
  opacity: 0;
  transition: opacity 0.5s ease-in-out;
  pointer-events: none;
}

.rejected .scram {
  opacity: 1;
}

.accepted .ok {
  opacity: 1;
}

.ok {
  left: 49px;
}

.scram {
  left: 55px;
}

h1, h2 {
  color: #333;
}

h1 {
  margin: 0;
  padding-top: 30px;
  font-size: 36px;
  text-align: center;
}

h2 {
  margin: 0;
  padding-top: 18px;
  font-size: 24px;
}

.subhead {
  text-align: center;
}

#slat-container {
  position: relative;
  top: 30px;
  width:80px;
  height:25px;
  background: black;
  margin: 0 auto;
  overflow: hidden;
}

#slat {
  position: absolute;
  top:0;
  left:0;
  width:80px;
  height:25px;
  background: white;
  box-sizing: border-box;
  border: 1px solid black;
  margin: 0 auto;
}

@keyframes knock {
  0% {background: white;}
  100% {background: gray;}
}

@keyframes check {
  0% {top: 0px;}
  10% {top: -25px;}
  30% {top: -25px;}
  40% {top: 0px;}
  100% {top: 0px;}
}

@keyframes scan {
  0% {left: -1px;}
  10% {left: -1px;}
  20% {left: 10px;}
  30% {left: -1px;}
  40% {left: -1px;}
  100% {left: -1px;}
}

@keyframes rejected {
  0% {top: 0px;}
  10% {top: -25px;}
  90% {top: -25px;}
  100% {top: 0px;}
}

@keyframes scrutinize {
  0% {left: 2px;}
  33% {left: 2px;}
  66% {left: 2px; top: 6px}
  100% {left: 2px; top: 6px}
}

@keyframes scowlleft {
  0% {transform: rotate(0);}
  33% {transform: rotate(30deg);}
  100% {transform: rotate(30deg);}
}

@keyframes scowlright {
  0% {transform: rotate(0);}
  33% {transform: rotate(-30deg);}
  100% {transform: rotate(-30deg);}
}

.brow {
  background: black;
  position: absolute;
  width: 15px;
  height: 5px;
  transform: rotate(0);
}

.brow.left {
  left: 15px;
}

.brow.right {
  right: 15px;
}

.eye {
  position: absolute;
  width: 15px;
  height: 15px;
  border-radius: 7px;
  background-color: white;
  top: 5px;
}
.eye.left {
  left: 15px;
}
.eye.right {
  right: 15px;
}
.eye .ball {
  position: absolute;
  width: 10px;
  height: 10px;
  border-radius: 5px;
  top: 3px;
  left: -1px;
  background: black;
  animation: scan 8s infinite;
  animation-delay: 2s;
}

#knob {
  position: absolute;
  top: 150px;
  left: 140px;
  background: white;
  border-radius: 9px;
  border: 1px solid black;
  width: 18px;
  height: 18px;
  box-shadow: #333 0px 2px 3px;
}

#mask {
  position: relative;
  height: 310px;
  width: 180px;
  margin: 0 auto;
  overflow: hidden;
}

#door {
  position: absolute;
  background: #fff;
  border: 1px solid black;
  width: 180px;
  height: 310px;
  transform-origin: left;
  transition: transform 0.5s ease-in-out;
}

#door.knocking {
  animation: knock .3s 1;
}

#door:focus {
  background: red;
}

#room {
  position: absolute;
  height: 310px;
  width: 180px;
  margin: 0 auto;
  background: #f0e337;
}

.accepted #door {
  transform: perspective(1200px) translateZ(0px) translateX(0px) translateY(0px) rotateY(+35deg);
}

.scanning #slat {
  animation: check 8s infinite;
  animation-delay: 2s;
}

.rejected #slat {
  animation: rejected 3s 1;
}

.rejected .eye .ball {
  animation: scrutinize 3s 1;
}

.rejected .brow.left {
  animation: scowlleft 3s 1;
}

.rejected .brow.right {
  animation: scowlright 3s 1;
}

.bubble {
  position: absolute;
  bottom: 0px;
  left: 50px;
  font-family: sans-serif;
  font-size: 16px;
  line-height: 24px;
  width: 100px;
  background: #ddd;
  border-radius: 10px;
  padding: 6px;
  text-align: center;
  color: #000;
}

.bubble-bottom-left:before {
  content: "";
  width: 0px;
  height: 0px;
  position: absolute;
  border-left: 12px solid #ddd;
  border-right: 6px solid transparent;
  border-top: 6px solid #ddd;
  border-bottom: 20px solid transparent;
  left: 32px;
  bottom: -24px;
}

#visualization-wrapper {
  margin-top: 10px;
}

#recording-visualization-wrapper {
  visibility: hidden;
  height: 0;
}

.recording #recording-visualization-wrapper {
  visibility: visible;
  height: auto;
}

.caption {
  text-align: center;
  font-style: italic;
  height: 0;
}

.recording .caption {
  height: auto;
}

</style>
