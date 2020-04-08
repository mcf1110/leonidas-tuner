<script>
  import pitchfinder from "pitchfinder";
  import teoria from "teoria";

  let pitch;
  let style = "";

  const detectPitch = pitchfinder.AMDF();
  navigator.mediaDevices
    .getUserMedia({ audio: true, video: false })
    .then(function(stream) {
      const audioCtx = new AudioContext();
      const source = audioCtx.createMediaStreamSource(stream);
      const processor = audioCtx.createScriptProcessor(1024, 1, 1);

      source.connect(processor);
      processor.connect(audioCtx.destination);

      processor.onaudioprocess = function(e) {
        // Do something with the data, i.e Convert this to WAV
        const float32Array = e.inputBuffer.getChannelData(0); // get a single channel of sound
        const p = detectPitch(float32Array);
        if (p) {
          pitch = teoria.note.fromFrequency(p);
        }
      };
    });
  let state = "NULL";
  $: {
    if (pitch) {
      state =
        Math.abs(pitch.cents) < 13 ? "OK" : pitch.cents < 0 ? "UP" : "DOWN";

      const rot = 90 + pitch.cents;
	  style = `transform: rotate(${rot}deg)`;
	  if (state == 'OK'){
		  style += '; background: white'
	  }
    }
  }
</script>

<style>

</style>

<main>
  <h1>Leônidas Tuner</h1>
  <p class="note">{pitch ? pitch.note.scientific() : ''}</p>
  <p class="cents">{pitch ? pitch.cents.toFixed(2) : ''}</p>

  <div class="leonidas">
    {#if state === 'OK'}
      <p>Ok</p>
      <img src="/ok.jpg" alt="Ok" />
    {:else if state === 'UP'}
      <p>Um pouco mais</p>
      <img src="/leonidas.png" alt="Um pouco mais" />
    {:else if state === 'DOWN'}
      <p>Um pouco menos</p>
      <img src="/leonidas.png" alt="Um pouco menos" class="mirror" />
    {:else}
      <p>Pronto para afinar</p>
    {/if}
  </div>

  <div class="pointer">
    <div class="handle" {style} />
  </div>
</main>
