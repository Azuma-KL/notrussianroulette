<script>
  import { cubicIn } from "svelte/easing";
  import { fade } from "svelte/transition";

  let debug = false
  let bullets = $state([1, 2, 3, 4, 5, 6])
  let tries = $state([])
  let chance = $state(1)
  let shuffleSt = $state(false)
  let text = $state("")
  let gamestate = $state(0)
  let revolverSvg
  let rotation = $state(0)
  let liveBullet
  let isDisabled = $state(false)
  let inputChange = $state(false)
  // let isChoiceMode = $state(false)
  // let choiceText1 = $state("")
  // let choiceText2 = $state("")
  let position = $state({cx: 75, cy: 28}) // 1
  let bulletPos = $state([
    {cx: 75, cy: 28},
    {cx: 34, cy: 51},
    {cx: 34, cy: 98.5},
    {cx: 75, cy: 122},
    {cx: 116, cy: 98.5},
    {cx: 116, cy: 51},
  ])

  function rotateSvg() { // 1
    rotation += 360
    revolverSvg.style.transform = `rotate(${rotation}deg)`
    revolverSvg.style.transition = `transform 1s ease-out`
  }
  function singleRotateSvg() { // 1
    rotation += 60
    revolverSvg.style.transform = `rotate(${rotation}deg)`
    revolverSvg.style.transition = `transform 0.1s ease-out`
  }

  function shuffle(array) {
    for (let i = array.length - 1; i > 0; i--) {
      let j = Math.floor(Math.random() * (i + 1)); // 1
      [array[i], array[j]] = [array[j], array[i]] 
    }
    rotateSvg()
    shuffleSt = true
    
    let valueToFind = 1; // 1
    let indexInChance = bullets.indexOf(valueToFind) // 1
    position = bulletPos[indexInChance]; // 1
  }
  

  function roll() {
    if(!tries.includes(1)){
      tries.push(bullets[0])
      bullets.splice(0, 1)
      singleRotateSvg() // 1
      if(tries.length == chance && !tries.includes(1)){
        gamestate = 1
        isDisabled = true
      } else if (tries.length <= chance && tries.includes(1)) {
        gamestate = 2
        isDisabled = true
        liveBullet.style.fill = 'tomato'
      }
    }
  }

  function reset() {
    bullets = [1, 2, 3, 4, 5, 6]
    position = {cx: 75, cy: 28}
    tries = []
    chance = 1
    shuffleSt = false
    gamestate = 0
    text = ""
    rotation = 0 
    inputChange = false
    isDisabled = false
    revolverSvg.style.transform = `rotate(${rotation}deg)`
    revolverSvg.style.transition = `transform`
    liveBullet.style.fill = '#D9D9D9'
    choiceText1 = ""
    choiceText2 = ""
  }

  function inputReset() {
    inputChange = true
  }
</script>

<!-- <div class="main"> -->
  <div class="container">
    <!-- <div
    class="resultBox"
    style="background-color: {gamestate == 1 ? 'mediumseagreen' : gamestate == 2 ? 'tomato' : 'transparent'}; border: {gamestate != 0 ? '2px solid black' : 'transparent'}"
  >
    {#if gamestate == 1}
      <span class="result">Win</span>
    {:else if gamestate == 2}
      <span class="result">Loss</span>
    {/if}
  </div> -->

  <!-- TODO: CHOICE MODE !!!!!!!!!!!!!!!
  {#if isChoiceMode == true}
    <span>What will you do:</span>
    <div class="choices">
      <input type="text" id="choiceInput" placeholder="If you survive" bind:value={choiceText1}>
      <input type="text" id="choiceInput" placeholder="If you die" bind:value={choiceText2}>
    </div>
  {/if} -->

  {#if gamestate == 1}
    <div class="resultBox" style="background-color: mediumseagreen;">
      <span class="result">You Survived</span>
    </div>
  {:else if gamestate == 2}
    <div class="resultBox" style="background-color: tomato;">
      <span class="result">You Died</span>
    </div>
  {/if}

  {#if shuffleSt == true}
    {#if tries.length == 5}
      <button id="endItButton" onclick={() => roll()} disabled={isDisabled}>End It All</button>
      <!-- in:fade={{ duration: 1000, delay: 250, easing: cubicIn }} -->
    {:else if gamestate == 2 && tries.length == 6}
      <button disabled={isDisabled}>End It All</button>
    {:else}
      <button onclick={() => roll()} disabled={isDisabled}>Roll</button>
    {/if}
    {#if debug}
    <span>Tries: {tries}</span>
    {/if} 
  {/if}

  {#if debug}
  <span>Chances: {bullets}</span>
  {/if} 

  {#if tries.length <= 0}
    <button onclick={() => shuffle(bullets)}>Shuffle</button>
    <span>How many rounds can you survive? {chance}</span>
    <input type="range" id="range" min=1 max=6 bind:value={chance} oninput={() => inputReset()}>
    {#if chance >= 5}
      <span>Woah there</span>
    {:else if chance >= 3}
      <span>Test your luck</span>
    {:else if chance >= 1 && inputChange == true}
      <span>You coward</span>
    {/if}
  {:else}
    {#if bullets.length >= 1 && gamestate == 0}
      <span>Will you survive {chance} rounds?</span>
    {:else if bullets.length <= 1 && gamestate == 0}
      <span>Will you survive {chance} round?</span>
    {/if}

  {/if}
  {#if tries.length == 5}
    <button id="giveUpButton" onclick={() => reset()}>Give Up</button>
    <!-- in:fade={{ duration: 1000, delay: 1000, easing: cubicIn }} -->
  {:else}
    <button onclick={() => reset()}>Reset</button>
  {/if}
  <span>{text}</span>
  </div>

  <div class="gun">
    {#if debug}
      <button onclick={() => rotateSvg()}>Rotate</button>
      <button onclick={() => singleRotateSvg()}>Single Rotation</button>
    {/if}
    <svg width="150" height="150" viewBox="0 0 150 150" fill="none" xmlns="http://www.w3.org/2000/svg">
      <g id="revolver" bind:this={revolverSvg}>
          <g id="bullets">
            {#each bulletPos as pos, i}  
            <!-- // 1 -->
              <circle
                id={"blank" + (i + 1)}
                cx={pos.cx}
                cy={pos.cy}
                r="19"
                fill="#D9D9D9"
              />
            {/each}
            <circle id="shape" cx={position.cx} cy={position.cy} r="19" fill="#D9D9D9" bind:this={liveBullet}/>
          </g>
          <g id="chamber" opacity="0.46">
              <path fill-rule="evenodd" clip-rule="evenodd" d="M75 0C75.9906 3.92683e-09 76.9767 0.0195352 77.958 0.0576172C78.5545 0.080749 79.1494 0.111455 79.7422 0.148438C86.2972 0.557698 92.6243 1.809 98.6162 3.79492C99.6328 12.9115 107.365 20 116.754 20C119.414 19.9999 121.941 19.4304 124.221 18.4072C136.427 29.0307 145.163 43.5374 148.505 60.0029C143.959 63.3222 141.006 68.6908 141.006 74.75C141.006 80.8478 143.997 86.2464 148.592 89.5605C145.317 106.206 136.535 120.875 124.221 131.593C121.941 130.57 119.414 130 116.754 130C107.365 130 99.6328 137.088 98.6162 146.205C92.6243 148.191 86.2972 149.442 79.7422 149.852C79.1494 149.889 78.5545 149.919 77.958 149.942C76.9767 149.98 75.9906 150 75 150C74.0091 150 73.0225 149.98 72.041 149.942C71.4445 149.919 70.8497 149.889 70.2568 149.852C63.7018 149.442 57.3747 148.191 51.3828 146.205C50.3661 137.089 42.6347 130 33.2461 130C30.5858 130 28.0586 130.57 25.7793 131.593C13.4652 120.875 4.68248 106.206 1.40723 89.5605C6.0023 86.2465 8.99414 80.8479 8.99414 74.75C8.99414 68.6905 6.04004 63.3222 1.49414 60.0029C4.83621 43.5374 13.573 29.0307 25.7793 18.4072C28.0586 19.4304 30.5858 19.9999 33.2461 20C42.6347 20 50.3661 12.9115 51.3828 3.79492C51.3932 3.79147 51.4036 3.78762 51.4141 3.78418C57.3967 1.80423 63.7132 0.557 70.2568 0.148438C70.8497 0.111446 71.4445 0.0807564 72.041 0.0576172C73.0225 0.01952 74.0091 0 75 0ZM75 66C70.0294 66 66 70.0294 66 75C66 79.9706 70.0294 84 75 84C79.9706 84 84 79.9706 84 75C84 70.0294 79.9706 66 75 66Z" fill="#D9D9D9"/>
              <path d="M106.442 143.11C105.315 143.631 104.173 144.126 103.017 144.592L103.018 144.591C103.595 144.358 104.17 144.118 104.74 143.871C105.311 143.624 105.879 143.371 106.442 143.11Z" fill="#D9D9D9"/>
          </g>
      </g>
    </svg>
    
    <!-- TODO: CHOICE MODE !!!!!!!!!!!!!!!
    <span id="choiceCheckbox">
      <input type="checkbox" bind:checked={isChoiceMode}>
      Choice Mode
    </span> -->

    
  </div>
<!-- </div> -->

<style>
  .resultBox {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 50px;
    width: 100%;
    border: 2px solid black;
    border-radius: 10px;
    margin-block: 10px;
    min-height: 50px;
  }
  .result {
    font-weight: bold;
  }

  /* TODO: CHOICEMODE
  .choices {
    display: flex;
    gap: 10px;
  }
  #choiceInput {
    width: 100%;
  } */

  /* .main{
    height: 100%;
    min-width: 100%;
    display: flex;
    gap: 20px;
    justify-content: center;
    padding: 25px;
    border: 1px solid black;
    border-radius: 15px;
  } */
  .container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 250px;
    gap: 5px;
    min-height: 250px;
  }
  .gun {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
  }
  #revolver { 
    /* 1 */
    transform-origin: center center; 
    transition: transform 0.3s ease-out;
  }
  /* #giveUpButton {
    border: 2px solid aquamarine;
    background-color: rgb(174, 212, 255);
  }
  #endItButton {
    border: 2px solid tomato;
    background-color: rgb(117, 20, 20);
  } */
</style>