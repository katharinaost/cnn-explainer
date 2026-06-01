<script>
	import ConvolutionAnimator from './ConvolutionAnimator.svelte';
  import { singleConv } from '../utils/cnn.js';
  import { createEventDispatcher } from 'svelte';

  export let input;
  export let kernel;
  export let dataRange;
  export let colorScale = d3.interpolateRdBu;
  export let isInputInputLayer = false;
  export let isExited = false;
  // export let output;
  
  const dispatch = createEventDispatcher();
	let stride = 1;
  const dilation = 1;
  var isPaused = false;

  // This model uses `same` padding (stride 1), so a conv keeps the input's
  // spatial size (e.g. 28x28 -> 28x28). The ConvolutionAnimator expects a
  // *pre-padded* input (its internal padding is 0), so we zero-pad the input by
  // (kernel - 1) / 2 here. Without this the animator would slide an unpadded
  // `valid` convolution and show a smaller output (28 -> 26), contradicting the
  // (28,28) -> (28,28) maps in the network diagram. The padding border is shown
  // as the outer ring of zero-valued cells on the Input matrix.
  function padMatrix(matrix, pad) {
    if (pad <= 0) { return matrix; }
    let size = matrix.length + 2 * pad;
    let padded = [];
    for (let r = 0; r < size; r++) { padded.push(new Array(size).fill(0)); }
    for (let r = 0; r < matrix.length; r++) {
      for (let c = 0; c < matrix[r].length; c++) {
        padded[r + pad][c + pad] = matrix[r][c];
      }
    }
    return padded;
  }

  const convPadding = Math.floor((kernel.length - 1) / 2);
  $: paddedInput = padMatrix(input, convPadding);
  var outputFinal = singleConv(padMatrix(input, convPadding), kernel, stride);
  $: if (stride > 0) {
    try {
      outputFinal = singleConv(paddedInput, kernel, stride);
    } catch {
      console.log("Cannot handle stride of " + stride);
    }
  }
  
  function handleClickPause() {
    isPaused = !isPaused;
  }

  function handleScroll() {
    let svgHeight = Number(d3.select('#cnn-svg').style('height').replace('px', '')) + 150;
    let scroll = new SmoothScroll('a[href*="#"]', {offset: -svgHeight});
    let anchor = document.querySelector(`#article-convolution`);
    scroll.animateScroll(anchor);
  }

  function handlePauseFromInteraction(event) {
    isPaused = event.detail.text;
  }

  function handleClickX() {
    isExited = true;
    dispatch('message', {
      text: isExited
    });
  }
</script>

<style>
  .control-pannel {
    display: flex;
    position: relative;
    flex-direction: column;
    align-items: center;
  }

  .buttons {
    cursor: pointer;
    position: absolute;
    top: 0px;
    right: 0px;
  }

  .control-button {
    color: gray;
    font-size: 15px;
    opacity: 0.4;
    cursor: pointer;
  }

  .control-button:not(:first-child) {
    margin-left: 5px;
  }

  .annotation {
    display: flex;
    align-items: center;
    justify-content: center;
    padding-left : 10px;
    font-size: 12px;
  }

  .annotation > img {
    width: 17px;
    margin-right: 5px;
  }

  .control-button:hover {
    opacity: 0.8;
  }

  .box {
    padding: 5px 15px 10px 15px;
  }

  .container {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
  }

  .title-text {
    font-size: 1.2em;
    font-weight: 500;
    color: #4a4a4a;
  }
</style>

{#if !isExited}
  <div class="container" id="detailview-container">

    <!-- old stride input -->
    <!-- <div class="columns is-mobile">
      <div class="column is-half is-offset-one-quarter">
        <div class="field is-grouped">
          <p class="control is-expanded">
            <input class="input" type="text" placeholder="Stride" bind:value={stride} />
          </p>
          <p class="control">
            <button class="button is-success" on:click={handleClickPause}>
              Toggle Movement
            </button>
          </p>
        </div>
      </div>
    </div> -->

    <div class="box">

      <div class="control-pannel">

        <div class="title-text">
          Convolution
        </div>

        <div class="buttons">
          <div class="control-button" on:click={handleScroll} title="Jump to article section">
            <i class="fas fa-info-circle"></i>
          </div>

          <div class="play-button control-button" on:click={handleClickPause} title="Play animation">
            {@html isPaused ?
              '<i class="fas fa-play-circle play-icon"></i>' :
              '<i class="fas fa-pause-circle"></i>'}
          </div>

          <div class="delete-button control-button" on:click={handleClickX} title="Close">
            <i class="fas control-icon fa-times-circle"></i>
          </div>
        </div>
      </div>

      <div class="container is-centered">
        <ConvolutionAnimator on:message={handlePauseFromInteraction}
          kernel={kernel} image={paddedInput} output={outputFinal}
          stride={stride} dilation={dilation} isPaused={isPaused}
          dataRange={dataRange} colorScale={colorScale}
          isInputInputLayer={isInputInputLayer} inputPadding={convPadding} />
      </div>

      <div class="annotation">
        <img src='PUBLIC_URL/assets/img/pointer.svg' alt='pointer icon'>
        <div class="annotation-text">
          <span style="font-weight:600">Hover over</span> the matrices to change kernel position.
        </div>
      </div>

    </div>
  </div>
{/if}