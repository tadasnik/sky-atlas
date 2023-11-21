<script>
  import { onMount } from "svelte";
  import { DateInput } from "date-picker-svelte";
  import { select, zoom } from "d3";
  import Sky from "$lib/components/Sky.svelte";
  import { SiderealTime } from "$lib/astronomy/astronomy.ts";

  let skycanvas;
  let zoomTransform = { k: 1, x: 0, y: 0 };
  let width;
  let height;
  let latitude = 51.6;
  let mag_th = 6;
  let time = new Date();
  onMount(() => {
    const interval = setInterval(() => {
      time = new Date();
    }, 1000 * 60);

    return () => {
      clearInterval(interval);
    };
  });
  // these automatically update when `time`
  // changes, because of the `$:` prefix

  $: zoomX = zoom()
    .scaleExtent([1, 5])
    .translateExtent([
      [0, 0],
      [width, height],
    ])
    .on("zoom", handleZoom);

  function handleZoom(e) {
    zoomTransform.k = e.transform.k;
    zoomTransform.x = e.transform.x;
    zoomTransform.y = e.transform.y;
  }

  const successCallback = (position) => {
    console.log(position);
  };

  const errorCallback = (error) => {
    console.log("there was ", error);
  };

  $: if (skycanvas) {
    select(skycanvas).call(zoomX);
  }
  // $: console.log("sidereal time : ", SiderealTime(time));
  $: localAngle = 15 * SiderealTime(time);
  // $: console.log("local angle : ", localAngle);
  // onMount(() => {
  //   console.log(
  //     "location ",
  //     navigator.geolocation.getCurrentPosition(successCallback, errorCallback)
  //   );
  // });
</script>

<div class="outer-wrapper">
  <div class="controls">
    <DateInput
      bind:value={time}
      format="yyyy-MM-dd HH:mm"
      timePrecision="minute"
    />
    <!-- <label> -->
    <!--   <input -->
    <!--     type="range" -->
    <!--     min="0" -->
    <!--     max="360" -->
    <!--     bind:value={longitude} -->
    <!--     step="0.01" -->
    <!--   /> -->
    <!--   <input type="number" bind:value={longitude} min="0" max="360" /> -->
    <!-- </label> -->

    <label>
      Observer latitude
      <input type="number" bind:value={latitude} min="-90" max="90" />
    </label>

    <label>
      Limiting star magnitude
      <input type="number" bind:value={mag_th} min="0" max="14" step="0.1" />
    </label>
  </div>
  <div
    class="map-container"
    bind:clientWidth={width}
    bind:clientHeight={height}
  >
    <Sky {width} {height} longitude={localAngle} {latitude} {mag_th} />
  </div>
</div>

<style>
  /*
    The wrapper div needs to have an explicit width and height in CSS.
    It can also be a flexbox child or CSS grid element.
    The point being it needs dimensions since the <LayerCake> element will
    expand to fill it.
  */
  .outer-wrapper {
    position: relative;
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 100%;
    height: 100%;
  }
  .controls {
    z-index: 300;
    width: 100%;
    padding: 0 0 1vh 0;
    text-align: center;
  }
  .map-container {
    width: 100%;
    height: 100%;
  }
</style>
