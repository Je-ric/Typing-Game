<template>
    <div class="min-h-screen bg-gradient-to-r from-[#5680E9] via-[#84CEEB] to-[#E9F1FB] flex flex-col items-center justify-center p-4">
  <!-- Game Container -->
  <div class="w-full max-w-3xl bg-white/80 rounded-lg shadow-lg p-6">
    <!-- Game Title -->
    <h1 class="text-4xl font-extrabold text-gray-800 mb-6">Vue Typing Master</h1>
  
        <!-- Game Menu (shown when not playing) -->
        <div v-if="gameState === 'menu'" class="space-y-6">
          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <!-- Difficulty Selection -->
            <div class="space-y-2">
              <h2 class="text-xl font-semibold text-gray-700">Difficulty</h2>
              <div class="flex flex-wrap gap-2">
                <button 
                  v-for="level in difficulties" 
                  :key="level.name"
                  @click="selectedDifficulty = level"
                  :class="[
                    'px-4 py-2 rounded-md transition-colors',
                    selectedDifficulty.name === level.name 
                      ? 'bg-primary text-white' 
                      : 'bg-gray-200 hover:bg-gray-300 text-gray-800'
                  ]"
                >
                  {{ level.name }}
                </button>
              </div>
            </div>
  
            <!-- Theme Selection -->
            <div class="space-y-2">
              <h2 class="text-xl font-semibold text-gray-700">Theme</h2>
              <div class="flex flex-wrap gap-2">
                <button 
                  v-for="theme in themes" 
                  :key="theme.name"
                  @click="selectedTheme = theme"
                  :class="[
                    'px-4 py-2 rounded-md transition-colors',
                    selectedTheme.name === theme.name 
                      ? 'bg-primary text-white' 
                      : 'bg-gray-200 hover:bg-gray-300 text-gray-800'
                  ]"
                >
                  {{ theme.name }}
                </button>
              </div>
            </div>
          </div>
  
          <!-- Game Duration -->
          <div class="space-y-2">
            <h2 class="text-xl font-semibold text-gray-700">Duration</h2>
            <div class="flex flex-wrap gap-2">
              <button 
                v-for="duration in durations" 
                :key="duration"
                @click="selectedDuration = duration"
                :class="[
                  'px-4 py-2 rounded-md transition-colors',
                  selectedDuration === duration 
                    ? 'bg-primary text-white' 
                    : 'bg-gray-200 hover:bg-gray-300 text-gray-800'
                ]"
              >
                {{ duration }} seconds
              </button>
            </div>
          </div>
  
          <!-- Start Game Button -->
          <button 
            @click="startGame" 
            class="w-full py-3 bg-primary hover:bg-primary-dark text-white font-bold rounded-md transition-colors"
          >
            Start Game
          </button>
        </div>
  
        <!-- Game Play (shown during gameplay) -->
        <div v-else-if="gameState === 'playing'" class="space-y-6">
          <!-- Stats Bar -->
          <div class="flex justify-between items-center bg-gray-100 p-3 rounded-md">
            <div class="text-gray-700">
              <span class="font-semibold">Time:</span> {{ timeLeft }}s
            </div>
            <div class="text-gray-700">
              <span class="font-semibold">WPM:</span> {{ currentWPM }}
            </div>
            <div class="text-gray-700">
              <span class="font-semibold">Accuracy:</span> {{ accuracy }}%
            </div>
          </div>
  
          <!-- Text Display -->
          <div class="bg-gray-50 p-4 rounded-md border border-gray-200 font-mono text-lg leading-relaxed">
            <p class="whitespace-pre-wrap">
              <span 
                v-for="(char, index) in currentText" 
                :key="index"
                :class="{
                  'bg-green-200 text-green-800': index < typedText.length && char === typedText[index],
                  'bg-red-200 text-red-800': index < typedText.length && char !== typedText[index],
                  'text-gray-400': index >= typedText.length
                }"
              >{{ char }}</span>
            </p>
          </div>
  
          <!-- Input Area -->
          <div class="relative">
            <textarea
              ref="textInput"
              v-model="typedText"
              @input="checkTyping"
              class="w-full p-4 border border-gray-300 rounded-md font-mono text-lg resize-none focus:outline-none focus:ring-2 focus:ring-primary focus:border-transparent"
              rows="4"
              :disabled="timeLeft <= 0"
              placeholder="Start typing here..."
              autofocus
            ></textarea>
          </div>
  
          <!-- Restart Button -->
          <button 
            @click="resetGame" 
            class="w-full py-3 bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold rounded-md transition-colors"
          >
            Restart
          </button>
        </div>
  
        <!-- Results Screen -->
        <div v-else-if="gameState === 'results'" class="space-y-6">
          <h2 class="text-2xl font-bold text-center text-gray-800">Game Results</h2>
          
          <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
            <div class="bg-gray-50 p-4 rounded-md border border-gray-200 text-center">
              <p class="text-gray-500 text-sm">Words Per Minute</p>
              <p class="text-3xl font-bold text-primary">{{ finalWPM }}</p>
            </div>
            
            <div class="bg-gray-50 p-4 rounded-md border border-gray-200 text-center">
              <p class="text-gray-500 text-sm">Accuracy</p>
              <p class="text-3xl font-bold text-primary">{{ finalAccuracy }}%</p>
            </div>
            
            <div class="bg-gray-50 p-4 rounded-md border border-gray-200 text-center">
              <p class="text-gray-500 text-sm">Score</p>
              <p class="text-3xl font-bold text-primary">{{ finalScore }}</p>
            </div>
          </div>
  
          <div class="space-y-2">
            <h3 class="text-xl font-semibold text-gray-700">Performance</h3>
            <div class="bg-gray-50 p-4 rounded-md border border-gray-200">
              <p class="text-gray-700">
                <span class="font-semibold">Characters typed:</span> {{ charsTyped }}
              </p>
              <p class="text-gray-700">
                <span class="font-semibold">Correct characters:</span> {{ correctChars }}
              </p>
              <p class="text-gray-700">
                <span class="font-semibold">Errors:</span> {{ errors }}
              </p>
            </div>
          </div>
  
          <!-- Action Buttons -->
          <div class="flex gap-4">
            <button 
              @click="gameState = 'menu'" 
              class="flex-1 py-3 bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold rounded-md transition-colors"
            >
              Main Menu
            </button>
            <button 
              @click="restartSameSettings" 
              class="flex-1 py-3 bg-primary hover:bg-primary-dark text-white font-bold rounded-md transition-colors"
            >
              Play Again
            </button>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onUnmounted, nextTick } from 'vue';
  
  // Game state
  const gameState = ref('menu'); // 'menu', 'playing', 'results'
  const timeLeft = ref(0);
  const typedText = ref('');
  const currentText = ref('');
  const timer = ref(null);
  const startTime = ref(0);
  const textInput = ref(null);
  
  // Game settings
  const difficulties = [
    { name: 'Easy', wordLength: 'short', textLength: 100 },
    { name: 'Medium', wordLength: 'medium', textLength: 150 },
    { name: 'Hard', wordLength: 'long', textLength: 200 }
  ];
  
  const themes = [
    { 
      name: 'Programming', 
      words: [
        'function', 'variable', 'const', 'let', 'class', 'object', 'method',
        'array', 'string', 'number', 'boolean', 'null', 'undefined', 'promise',
        'async', 'await', 'component', 'props', 'state', 'vue', 'react', 'angular',
        'javascript', 'typescript', 'python', 'java', 'code', 'algorithm', 'data',
        'structure', 'interface', 'implementation', 'compiler', 'interpreter'
      ]
    },
    { 
      name: 'Nature', 
      words: [
        'mountain', 'river', 'ocean', 'forest', 'tree', 'flower', 'animal',
        'bird', 'fish', 'insect', 'butterfly', 'eagle', 'lion', 'tiger',
        'elephant', 'dolphin', 'whale', 'coral', 'reef', 'desert', 'jungle',
        'savanna', 'tundra', 'glacier', 'volcano', 'earthquake', 'hurricane',
        'climate', 'weather', 'rain', 'snow', 'wind', 'sunshine', 'cloud'
      ]
    },
    { 
      name: 'Food', 
      words: [
        'pizza', 'pasta', 'burger', 'sandwich', 'salad', 'soup', 'steak',
        'chicken', 'fish', 'vegetable', 'fruit', 'apple', 'banana', 'orange',
        'grape', 'strawberry', 'blueberry', 'chocolate', 'vanilla', 'dessert',
        'cake', 'cookie', 'ice cream', 'coffee', 'tea', 'juice', 'water',
        'milk', 'cheese', 'bread', 'rice', 'noodle', 'spice', 'herb'
      ]
    }
  ];
  
  const durations = [30, 60, 120];
  
  const selectedDifficulty = ref(difficulties[0]);
  const selectedTheme = ref(themes[0]);
  const selectedDuration = ref(durations[1]);
  
  // Game statistics
  const charsTyped = ref(0);
  const correctChars = ref(0);
  const errors = ref(0);
  const finalWPM = ref(0);
  const finalAccuracy = ref(0);
  const finalScore = ref(0);
  
  // Computed properties
  const currentWPM = computed(() => {
    if (typedText.value.length === 0) return 0;
    
    const timeElapsed = (Date.now() - startTime.value) / 1000 / 60; // in minutes
    const words = typedText.value.length / 5; // standard: 5 chars = 1 word
    return Math.round(words / timeElapsed);
  });
  
  const accuracy = computed(() => {
    if (typedText.value.length === 0) return 100;
    
    let correct = 0;
    for (let i = 0; i < typedText.value.length; i++) {
      if (i < currentText.value.length && typedText.value[i] === currentText.value[i]) {
        correct++;
      }
    }
    
    return Math.round((correct / typedText.value.length) * 100);
  });
  
  // Game functions
  function generateText() {
    const { words } = selectedTheme.value;
    const { textLength } = selectedDifficulty.value;
    
    let result = '';
    let currentLength = 0;
    
    while (currentLength < textLength) {
      const randomWord = words[Math.floor(Math.random() * words.length)];
      result += randomWord + ' ';
      currentLength += randomWord.length + 1;
    }
    
    return result.trim();
  }
  
  function startGame() {
    gameState.value = 'playing';
    timeLeft.value = selectedDuration.value;
    typedText.value = '';
    currentText.value = generateText();
    startTime.value = Date.now();
    
    // Start the timer
    timer.value = setInterval(() => {
      timeLeft.value--;
      if (timeLeft.value <= 0) {
        endGame();
      }
    }, 1000);
    
    // Focus the text input
    nextTick(() => {
      if (textInput.value) {
        textInput.value.focus();
      }
    });
  }
  
  function checkTyping() {
    // Calculate statistics in real-time
    let correct = 0;
    for (let i = 0; i < typedText.value.length; i++) {
      if (i < currentText.value.length && typedText.value[i] === currentText.value[i]) {
        correct++;
      }
    }
    
    correctChars.value = correct;
    errors.value = typedText.value.length - correct;
    
    // Check if the user has completed the text
    if (typedText.value.length >= currentText.value.length) {
      endGame();
    }
  }
  
  function endGame() {
    clearInterval(timer.value);
    
    // Calculate final statistics
    charsTyped.value = typedText.value.length;
    finalWPM.value = currentWPM.value;
    finalAccuracy.value = accuracy.value;
    
    // Calculate score based on WPM, accuracy and difficulty
    const difficultyMultiplier = 
      selectedDifficulty.value.name === 'Easy' ? 1 :
      selectedDifficulty.value.name === 'Medium' ? 1.5 : 2;
    
    finalScore.value = Math.round(finalWPM.value * (finalAccuracy.value / 100) * difficultyMultiplier);
    
    // Change game state to results
    gameState.value = 'results';
  }
  
  function resetGame() {
    clearInterval(timer.value);
    gameState.value = 'menu';
    typedText.value = '';
    timeLeft.value = 0;
  }
  
  function restartSameSettings() {
    startGame();
  }
  
  // Cleanup on component unmount
  onUnmounted(() => {
    if (timer.value) {
      clearInterval(timer.value);
    }
  });
  </script>
  
  <style>
  :root {
    --color-primary: #116466;
    --color-primary-dark: #D1E8E2;
  }
  
  .bg-primary {
    background-color: var(--color-primary);
  }
  
  .bg-primary-dark {
    background-color: var(--color-primary-dark);
  }
  
  .text-primary {
    color: var(--color-primary);
  }
  
  .focus\:ring-primary:focus {
    --tw-ring-color: var(--color-primary);
  }
  
  .hover\:bg-primary-dark:hover {
    background-color: var(--color-primary-dark);
  }
  </style>
  