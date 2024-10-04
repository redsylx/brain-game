<script lang="ts">
  import { onMount } from "svelte";
  import { writable } from "svelte/store";

  type Question = {
      num1: number;
      num2: number;
  };

  type Stats = {
      correctAnswers: number;
      timeRecords: number[];
      totalQuestions: number;
  };

  let numberOfGames: number = 0;
  let currentQuestionIndex: number = 0;
  let currentAnswer: string = "";
  let correctAnswers: number = 0;
  let questions: Question[] = [];
  let timeRecords: number[] = [];
  let startTime: number | null = null;

  const gameStarted = writable<boolean>(false);

  function generateQuestions(): void {
      questions = Array.from({ length: numberOfGames }, (): Question => {
          const num1 = Math.floor(Math.random() * 9) + 11; // Random 2-digit number
          const num2 = Math.floor(Math.random() * 9) + 11; // Random 2-digit number
          return { num1, num2 };
      });
  }

  function startGame(): void {
      if (numberOfGames > 0) {
          generateQuestions();
          currentQuestionIndex = 0;
          correctAnswers = 0;
          timeRecords = [];
          startTime = Date.now();
          gameStarted.set(true);
      }
  }

  function submitAnswer(): void {
      if (questions[currentQuestionIndex]) {
          const currentQuestion = questions[currentQuestionIndex];
          const correctAnswer: number = currentQuestion.num1 * currentQuestion.num2;

          if (parseInt(currentAnswer) === correctAnswer) {
              correctAnswers++;
          }

          const endTime: number = Date.now();
          if (startTime !== null) {
              timeRecords.push((endTime - startTime) / 1000);
          }
          startTime = endTime;

          currentAnswer = "";
          currentQuestionIndex++;

          if (currentQuestionIndex >= questions.length) {
              gameStarted.set(false);
              saveStatistics();
          }
      }
  }

  function saveStatistics(): void {
      const stats: Stats = {
          correctAnswers,
          timeRecords,
          totalQuestions: questions.length,
      };
      localStorage.setItem("gameStats", JSON.stringify(stats));
  }

  function handleKeyPress(event: KeyboardEvent): void {
    if (event.key === 'Enter') {
        submitAnswer();
    }
  }

  onMount(() => {
      const savedStats = localStorage.getItem("gameStats");
      if (savedStats) {
          console.log("Previous Game Stats: ", JSON.parse(savedStats) as Stats);
      }
  });
</script>

<main class="flex flex-col items-center justify-center min-h-screen bg-gray-100 p-4">
  <div class="w-full max-w-md bg-white shadow-md rounded-lg p-6">
      {#if $gameStarted}
          <div>
              <h2 class="text-2xl font-bold mb-4">Question {currentQuestionIndex + 1} of {numberOfGames}</h2>
              <p class="text-lg mb-4">
                  {questions[currentQuestionIndex].num1} x {questions[currentQuestionIndex].num2} = ?
              </p>
              <input
                  type="number"
                  bind:value={currentAnswer}
                  placeholder="Enter your answer"
                  class="w-full p-2 border border-gray-300 rounded mb-4"
                  on:keydown={handleKeyPress}
              />

              <button
                  on:click={submitAnswer}
                  class="w-full bg-blue-500 text-white p-2 rounded"
              >
                  Submit Answer
              </button>
          </div>
      {:else}
          <div>
              <h1 class="text-3xl font-bold mb-6 text-center">Multiplication Game</h1>
              <label class="block mb-2">Number of Games:</label>
              <input
                  type="number"
                  bind:value={numberOfGames}
                  placeholder="Enter number of games"
                  class="w-full p-2 border border-gray-300 rounded mb-4"
              />
              <button
                  on:click={startGame}
                  class="w-full bg-green-500 text-white p-2 rounded"
              >
                  Start Game
              </button>
          </div>
      {/if}

      {#if !$gameStarted && currentQuestionIndex > 0}
          <div class="mt-6">
              <h2 class="text-2xl font-bold mb-4">Game Over</h2>
              <p>Total Questions: {questions.length}</p>
              <p>Correct Answers: {correctAnswers}</p>
              <p class="mb-4">Time Taken for Each Question (seconds):</p>
              <ul class="list-disc pl-5">
                  {#each timeRecords as time, index}
                      <li>Question {index + 1}: {time} seconds</li>
                  {/each}
              </ul>
          </div>
      {/if}
  </div>
</main>

<style>
  main {
      font-family: 'Arial', sans-serif;
  }
</style>
