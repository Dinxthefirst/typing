<script lang="ts">
  import { onMount } from "svelte";

  const emptyKey = "\u00A0";
  const alphabet = "abcdefghijklmnopqrstuvwxyz";
  const keyboardRows = ["qwertyuiop", "asdfghjkl", "zxcvbnm" + emptyKey];
  let startTime: number;
  let timeBetweenCorrectLetters = $state(0);
  let letterTimer = $state(1000);
  let currentLetter = $state("");
  let pressedKeys: string[] = $state([]);
  let score = $state(0);
  let gameStarted = $state(false);

  function handleKeydown(event: KeyboardEvent) {
    const key = event.key.toLowerCase();
    if (!alphabet.includes(key)) {
      return;
    }

    pressedKeys.push(key);

    if (currentLetter === emptyKey) {
      return;
    }

    if (key === currentLetter) {
      score++;
      timeBetweenCorrectLetters = (Date.now() - startTime) / score;
    }

    if (letterTimer != 0) {
      setTimeout(nextLetter, letterTimer);
    } else {
      nextLetter();
    }
  }

  function handleKeyup(event: KeyboardEvent) {
    const key = event.key.toLowerCase();
    pressedKeys = pressedKeys.filter((k) => k !== key);
  }

  function startGame() {
    gameStarted = true;
    startTime = Date.now();
    timeBetweenCorrectLetters = 0;
    nextLetter();
  }

  function endGame() {
    score = 0;
    gameStarted = false;
  }

  function restartGame() {
    endGame();
    startGame();
  }

  function nextLetter() {
    const randomIndex = Math.floor(Math.random() * alphabet.length);
    currentLetter = alphabet[randomIndex];
  }

  onMount(() => {
    window.addEventListener("keydown", handleKeydown);
    window.addEventListener("keyup", handleKeyup);

    return () => {
      window.removeEventListener("keydown", handleKeydown);
      window.removeEventListener("keyup", handleKeyup);
    };
  });

  function showLetterTimerTip(event: MouseEvent) {
    console.log("showLetterTimerTip");
    const tooltip = document.querySelector(".tooltip") as HTMLElement;
    tooltip.style.display = "block";
    tooltip.style.left = `${event.pageX + 10}px`;
    tooltip.style.top = `${event.pageY + 10}px`;
  }

  function hideLetterTimerTip() {
    const tooltip = document.querySelector(".tooltip") as HTMLElement;
    tooltip.style.display = "none";
  }
</script>

{#if !gameStarted}
  <div id="menu-container">
    <button id="start-button" onclick={startGame}>Start Game</button>
    <div id="slider-container">
      <div
        class="info"
        onmouseout={hideLetterTimerTip}
        onmousemove={showLetterTimerTip}
      >
        i
      </div>
      <div class="tooltip" style="display: none;">
        Change how often the letters will appear in milliseconds
      </div>
      <input
        type="number"
        id="letter-timer-input"
        min="0"
        max="10000"
        step="25"
        bind:value={letterTimer}
      />
      <input
        type="range"
        id="letter-timer"
        min="0"
        max="10000"
        step="25"
        bind:value={letterTimer}
      />
    </div>
  </div>
{:else}
  <div id="button-container">
    <button id="restart-button" onclick={restartGame}>Restart Game</button>
    <div>{timeBetweenCorrectLetters} ms</div>
    <button id="main-menu-button" onclick={endGame}>Main Menu</button>
  </div>
  <div id="score">Score: {score}</div>
  <div id="current-letter">{currentLetter.toUpperCase() || emptyKey}</div>
  <div id="keyboard">
    {#each keyboardRows as row}
      <div class="keyboard-row">
        {#each row.split("") as key}
          <div
            class="keyboard-key"
            class:pressed={pressedKeys.includes(key)}
            class:empty-key={key === emptyKey}
          >
            {key.toUpperCase()}
          </div>
        {/each}
      </div>
    {/each}
  </div>
{/if}

<style>
  @import url("https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap");

  * {
    margin: 0;
    font-family: "Roboto", sans-serif;
    font-size: 1.5rem;
  }

  button {
    padding: 1rem 2rem;
    font-size: 1.5rem;
    background-color: #f0f0f0;
    border: none;
    cursor: pointer;
  }

  #menu-container {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    height: 100vh;
    gap: 3rem;
  }

  #start-button:hover {
    background-color: #e0e0e0;
  }

  #slider-container {
    display: flex;
    gap: 1rem;
  }

  #letter-timer-input {
    width: 5rem;
    text-align: center;
    appearance: textfield;
    border: 2px solid #f0f0f0;
  }

  #letter-timer {
    width: 10rem;
  }

  #button-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-direction: row;
    gap: 2rem;
  }

  #score {
    display: flex;
    justify-content: center;
    align-items: center;
    top: 1rem;
  }

  #current-letter {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 3rem;
    padding: 1rem;
  }

  #keyboard {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .info {
    background-color: #f0f0f0;
    border-radius: 50%;
    width: 2rem;
    height: 2rem;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: default;
  }

  .tooltip {
    position: absolute;
    background-color: #f0f0f0;
    width: 20rem;
    padding: 0.5rem;
    border-radius: 0.5rem;
    pointer-events: none;
    box-shadow: 0 0 0.5rem rgba(0, 0, 0, 0.25);
  }

  .keyboard-row {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
  }

  .keyboard-key.pressed {
    background-color: #f0f0f0;
  }

  .keyboard-key {
    display: flex;
    border: #f0f0f0 solid 1px;
    justify-content: center;
    align-items: center;
    height: 2.5rem;
    width: 2.5rem;
    margin: 0.1rem;
  }

  .keyboard-key.empty-key {
    border: none;
  }
</style>
