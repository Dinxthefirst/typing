<script lang="ts">
  import { onMount } from "svelte";

  const alphabet = "abcdefghijklmnopqrstuvwxyz";
  const keyboardRows = ["qwertyuiop", "asdfghjkl", "zxcvbnm "];
  let currentLetter = "";
  let pressedKeys: string[] = [];
  let score = 0;
  let gameStarted = false;
  const letterTimer = 5000;

  function handleKeydown(event: KeyboardEvent) {
    const key = event.key.toLowerCase();
    if (!alphabet.includes(key)) {
      return;
    }

    pressedKeys = [...pressedKeys, key];

    key === currentLetter ? score++ : score--;
    currentLetter = "";
  }

  function handleKeyup(event: KeyboardEvent) {
    const key = event.key.toLowerCase();
    pressedKeys = pressedKeys.filter((k) => k !== key);
  }

  function startGame() {
    gameStarted = true;
  }

  onMount(() => {
    window.addEventListener("keydown", handleKeydown);
    window.addEventListener("keyup", handleKeyup);

    const interval = setInterval(() => {
      if (!gameStarted) {
        return;
      }
      const index = Math.floor(Math.random() * alphabet.length);
      currentLetter = alphabet[index];
    }, letterTimer);

    return () => {
      window.removeEventListener("keydown", handleKeydown);
      window.removeEventListener("keyup", handleKeyup);
      clearInterval(interval);
    };
  });
</script>

{#if !gameStarted}
  <button on:click={startGame}>Start Game</button>
{:else}
  <div id="score">Score: {score}</div>
  <div id="current-letter">{currentLetter.toUpperCase() || "\u00A0"}</div>
  <div id="keyboard">
    {#each keyboardRows as row}
      <div class="keyboard-row">
        {#each row.split("") as key}
          <div
            class="keyboard-key"
            class:pressed={pressedKeys.includes(key)}
            class:empty-key={key === " "}
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
