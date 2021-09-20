<script>

  import { Prohibition } from 'prohibition'
  console.log(Prohibition)
  let open = false;
  function toggleOpen() {
    open = !open
  }
</script>

<main>
  <div class="wall">
    <h1>PROHIBITION</h1>
    <p class="subhead">Secret Knock Authentication</p>
    <div class="darkness"></div>
      <div class="door-frame">
        <div class="back-door">
          <div class="door knocking" class:door-open="{open}" on:click={toggleOpen}>
            <div class="slat" />
            <div class="handle" />
          </div>
        </div>
      </div>
  </div>
  <div class="bubble bubble-bottom-left">Get Lost!</div>
</main>

<style>


h1 {
  font-family: 'Rampart One', cursive;
  z-index: 5000;
  color: #aaa;
  text-align: center;
  padding-top: 30px;
  margin: 0;
  font-size: 64px;
  position: relative;

}

@media only screen and (max-width: 1624px) {
  h1 {
    font-size: 36px;
  }
}

.subhead {
  position: relative;
  z-index: 5000;
  text-align: center;
  color: white;
}

.darkness {
  position: absolute;
  pointer-events: none;
  z-index: 1;
  margin: 0;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  width: 100%;
  height: 100%;
  background-image: radial-gradient(circle, transparent 0%, black 80%),
  radial-gradient(ellipse at top, black 0%, transparent 50%),
  radial-gradient(ellipse at 33% 0%, black 0%, transparent 50%),
  radial-gradient(ellipse at 66% 0%, black 0%, transparent 50%);
}

.wall {
  z-index: 0;
  position: relative;
  background-color: white;
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}

.back-door {
  z-index: 1;
  background-color: #111;
  position: relative;
  border-radius: 2px;
  width: 180px;
  height: 310px;
  margin: 0 auto;
  margin-top:50px;
}

.knocking {
  border: 1px solid red;
}

.door-frame {
  position: relative;
  width: 200px;
  height: 315px;
  margin: 0 auto;
  background: #363636;
}

.door {
  background-color: #444;
  position: absolute;
  border-radius: 2px;
  top: 0px;
  left: 0px;
  width: 180px;
  height: 310px;
  transform-origin: left;
  transition: all 0.5s ease-in-out;
}

.slat {
  position: absolute;
  left: 70px;
  top: 50px;
  width: 60px;
  height: 25px;
  background-color: #333;
}

.handle {
  position: absolute;
  left: 140px;
  top: 135px;
  width: 20px;
  height: 20px;
  border-radius: 20px;
  background-color: black;
}

.door-open {
  transform: perspective(1200px) translateZ(0px) translateX(0px) translateY(0px) rotateY(-35deg);

}

.bubble {
  position: relative;
  font-family: sans-serif;
  font-size: 16px;
  line-height: 24px;
  width: 100px;
  background: #fff;
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
  border-left: 12px solid #fff;
  border-right: 6px solid transparent;
  border-top: 6px solid #fff;
  border-bottom: 20px solid transparent;
  left: 32px;
  bottom: -24px;
}

</style>
