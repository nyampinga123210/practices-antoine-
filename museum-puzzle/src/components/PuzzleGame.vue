<template>
  <!-- 
    Icyitonderwa: Conteneur nkuru y'umukino
    Ifite class dark-theme iyo umukoresha yahisemo mode yijimye
  -->
  <div class="game-container" :class="{ 'dark-theme': darkMode }">
    
    <!-- ==================== START SCREEN ==================== -->
    <!-- Iyi ni screen ibanza igaragara mbere y'uko umukino utangira -->
    <div v-if="!gameStarted" class="start-screen">
      <div class="start-card">
        <h1>🏛️ Rwandan Museum Puzzle</h1>
        <p>Explore Rwanda's history through sliding puzzles!</p>
        
        <div class="start-options">
          <!-- Buto yo gutangira umukino -->
          <button class="start-btn" @click="startGame">
            🎮 Start Game
          </button>
          
          <!-- Buto yo guhindura mode (umuco / ijimye) -->
          <button class="theme-btn" @click="toggleTheme">
            {{ darkMode ? '☀️ Light Mode' : '🌙 Dark Mode' }}
          </button>
          
          <!-- Buto yo gukoresha / guhagarika amajwi -->
          <button class="sound-btn" @click="toggleSound">
            {{ soundEnabled ? '🔊 Sound On' : '🔇 Sound Off' }}
          </button>
        </div>
        
        <!-- Amabwiriza y'ukuri umukino -->
        <div class="instructions">
          <h3>How to Play</h3>
          <p>• Click on tiles next to the empty space to move them</p>
          <p>• Arrange the pieces to complete the historical image</p>
          <p>• Each level reveals a fact about Rwandan history</p>
          <p>• Complete faster with fewer moves for a higher score!</p>
        </div>
      </div>
    </div>

    <!-- ==================== MAIN GAME ==================== -->
    <!-- Iyi ni screen nkuru y'umukino igaragaza puzzle -->
    <div v-else class="container">
      
      <!-- Header: Irumuri rugaragaza amanota, level, n'igihe -->
      <div class="game-header">
        <h1>🏛️ Rwandan Museum Puzzle</h1>
        
        <!-- Ibi ni stat-4 zigaragaza amakuru y'ingenzi y'umukino -->
        <div class="stats">
          <div class="stat-card">
            <span class="stat-label">Level</span>
            <span class="stat-value">{{ currentLevel + 1 }} / 10</span>
          </div>
          
          <div class="stat-card">
            <span class="stat-label">Score</span>
            <span class="stat-value">{{ score }}</span>
          </div>
          
          <div class="stat-card">
            <span class="stat-label">Moves</span>
            <span class="stat-value">{{ moveCount }}</span>
          </div>
          
          <div class="stat-card">
            <span class="stat-label">⏱️ Time</span>
            <span class="stat-value">{{ formattedTime }}</span>
          </div>
        </div>
      </div>

      <!-- Progress Bar: Iragaragaza intambwe ugeze muri level 10 -->
      <div class="progress-section">
        <div class="progress-label">
          <span>Level Progress</span>
          <span>{{ currentLevel + 1 }} / 10</span>
        </div>
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: ((currentLevel + 1) / 10 * 100) + '%' }"></div>
        </div>
      </div>

      <!-- Game Board: Aha niho puzzle igaragara -->
      <div class="board-wrapper">
        <div class="board">
          <!-- Iterate through tiles (ibice 9 bya puzzle) -->
          <div
            v-for="(tile, index) in tiles"
            :key="index"
            class="tile"
            :class="{ empty: tile === 0, 'move-animation': lastMoveIndex === index }"
            @click="moveTile(index)"
          >
            <!-- Iyo tile ari 0 (ubusa) ntikigaragara -->
            <div
              v-if="tile !== 0"
              class="piece"
              :style="getTileStyle(tile)"
            >
              <!-- Numero igaragara kuri buri tile kugirango umukoresh yoroherezwe -->
              <span class="number">{{ tile }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Buto z'ubuyobozi: Try Again, Theme, Sound Toggle -->
      <div class="buttons">
        <button @click="shuffleTiles" class="btn btn-primary">
          🔄 Try Again
        </button>
        <button @click="toggleTheme" class="btn btn-secondary">
          {{ darkMode ? '☀️' : '🌙' }}
        </button>
        <button @click="toggleSound" class="btn btn-secondary">
          {{ soundEnabled ? '🔊' : '🔇' }}
        </button>
      </div>

      <!-- Historical Information Card: Iragaragaza amateka y'u Rwanda -->
      <div v-if="currentLevelFact" class="history-card">
        <div class="history-icon">📜</div>
        <div class="history-content">
          <h3>Historical Figure</h3>
          <p>{{ currentLevelFact }}</p>
        </div>
      </div>

      <!-- Trophy Screen: Iragaragaza iyo umukino urangiye (Victory Modal) -->
      <div v-if="won" class="modal-overlay">
        <div class="trophy-screen">
          <div class="trophy-icon">🏆</div>
          <h2>Congratulations!</h2>
          
          <!-- Ibi bitaramo bigaragaza amakuru y'iyi level urangiye -->
          <div class="level-stats">
            <p>✨ Time: {{ formattedTime }}</p>
            <p>🎯 Moves: {{ moveCount }}</p>
            <p>⭐ Score Gained: {{ lastLevelScore }}</p>
          </div>
          
          <!-- Fun Fact: Amakuru y'amateka y'u Rwanda -->
          <div class="fun-fact">
            <h3>📖 Fun Fact</h3>
            <p>{{ currentFact }}</p>
          </div>
          
          <!-- Buto yo gukomeza kuri level ikurikira (niba hari) -->
          <button 
            v-if="currentLevel < 9" 
            @click="nextLevel" 
            class="btn btn-success"
          >
            🔓 Unlock Next Picture
          </button>
          
          <!-- Buto yo kurangiza game iyo uri kuri level ya 10 -->
          <button 
            v-else-if="currentLevel === 9 && !gameCompleted" 
            @click="completeGame" 
            class="btn btn-success"
          >
            🏁 Complete Game
          </button>
          
          <!-- Ibi bigaragara iyo game yose irangiye (10 levels) -->
          <div v-if="gameCompleted" class="completion-screen">
            <h2>🏆 You Completed All 10 Levels! 🏆</h2>
            <p>Final Score: {{ score }}</p>
            <p>Total Time: {{ totalFormattedTime }}</p>
            <p>Total Moves: {{ totalMoves }}</p>
            <button @click="resetGame" class="btn btn-primary">
              🎮 Play Again
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// ============================================================
// IBIKORWA BYO KWINJIZA AMAFOTO (IMAGES)
// Aya mafoto ni yo akoreshwa kuri puzzle buri level
// ============================================================
import picture1 from '../assets/picture1.jpg'
import picture2 from '../assets/picture2.jpg'
import picture3 from '../assets/picture3.jpg'
import picture4 from '../assets/picture4.jpg'
import picture5 from '../assets/picture5.jpg'
import picture6 from '../assets/picture6.jpg'
import picture7 from '../assets/picture7.jpg'
import picture8 from '../assets/picture8.jpg'
import picture9 from '../assets/picture9.jpg'
import picture10 from '../assets/picture10.jpg'

export default {
  name: 'PuzzleGame',

  // ============================================================
  // DATA: Iyi ni data igizwe n'ibintu byose umukino ukoresha
  // ============================================================
  data() {
    return {
      // -------------------- STATE Y'UMUKINO --------------------
      gameStarted: false,      // Umukino watangiye? (true/false)
      gameCompleted: false,    // Level 10 zarangiye? (true/false)
      won: false,              // Iyi level irangiye? (true/false)
      darkMode: false,         // Mode yijimye? (true/false)
      soundEnabled: true,      // Amajwi akoreshwa? (true/false)
      
      // -------------------- AMANOTA N'INTAMBWE --------------------
      score: 0,                // Amanota yose umukoresh yabonye
      currentLevel: 0,        // Level uriho (0-9, ariko ukoresha 1-10)
      moveCount: 0,           // Umubare w'uburyo ukanditse tiles
      lastLevelScore: 0,      // Amanota wabonye kuri level urangiye
      
      // -------------------- IGIHE (TIMER) --------------------
      timer: null,            // Timer ID yo guhagarika
      elapsedSeconds: 0,      // Amasegonda yamaze kuri iyi level
      levelStartTime: null,    // Igihe level yatangiye (timestamp)
      totalElapsedSeconds: 0,  // Igihe cyose umukino wamaze (level zose)
      
      // -------------------- PUZZLE TILES --------------------
      tiles: [1, 2, 3, 4, 5, 6, 7, 8, 0],  // Ibice 9 bya puzzle (0 = ubusa)
      lastMoveIndex: null,                  // Indome ya tile yanyuze (kuri animation)
      
      // -------------------- AMAFOTO N'AMATEGA --------------------
      // Amafoto ya buri level (10 pictures)
      images: [picture1, picture2, picture3, picture4, picture5, picture6, picture7, picture8, picture9, picture10],
      
      // Amateka y'u Rwanda ya buri level
      facts: [
        "King Kigeli V Ndahindurwa was the last king of Rwanda, ruling briefly in 1959-1961 before the monarchy was abolished.",
        "Agathe Uwilingiyimana was Rwanda's first female Prime Minister, serving in 1993 before her tragic assassination.",
        "Mutara III Rudahigwa was a beloved king who converted to Catholicism and helped modernize Rwanda in the 1930s-1950s.",
        "Alexis Kagame was a famous Rwandan historian, philosopher, and priest who preserved Rwandan oral traditions.",
        "Rosalie Gicanda was the last queen of Rwanda, remembered for her dignity and tragic death during the 1994 genocide.",
        "Ruganzu Ndoli is remembered as a legendary king from the 16th century who expanded the Rwandan kingdom.",
        "Felicite Niyitegeka is remembered for her courage in saving lives during the 1994 genocide before being martyred.",
        "Kanjogera played a major role in the royal court as the queen mother in the late 19th and early 20th century.",
        "Yuhi V Musinga was one of Rwanda's kings who ruled from 1896 to 1931 during the German and Belgian colonial period.",
        "Historical figures help us understand Rwanda's past - the country has over 10 documented kings in its monarchy history."
      ],
      
      // Amateka agaragara mu gihe (current fact)
      currentFact: "",
      currentLevelFact: ""
    }
  },

  // ============================================================
  // COMPUTED: Ibi ni properties zibarwa mu buryo bwihuse
  // ============================================================
  computed: {
    // Ibi bigabanya igihe mu minota n'amasegonda (mm:ss)
    formattedTime() {
      const minutes = Math.floor(this.elapsedSeconds / 60)
      const seconds = this.elapsedSeconds % 60
      return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
    },
    
    // Ibi bigabanya igihe cyose cya game (mm:ss)
    totalFormattedTime() {
      const minutes = Math.floor(this.totalElapsedSeconds / 60)
      const seconds = this.totalElapsedSeconds % 60
      return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
    },
    
    // Umubare w'uburyo bwakorewe (moves)
    totalMoves() {
      return this.moveCount
    }
  },

  // ============================================================
  // MOUNTED: Ibi bikora mugihe component imaze gushyirwa
  // ============================================================
  mounted() {
    // Reba niba hari preference y'umukoresh yabitswe kuri mode (umuco/ijimye)
    const savedTheme = localStorage.getItem('puzzle-theme')
    if (savedTheme === 'dark') {
      this.darkMode = true
    }
    
    // Reba niba hari preference y'umukoresh yabitswe kuri sound (on/off)
    const savedSound = localStorage.getItem('puzzle-sound')
    if (savedSound !== null) {
      this.soundEnabled = savedSound === 'true'
    }
    
    // Tangiza amajwi (preload)
    this.preloadAudio()
  },

  // ============================================================
  // BEFOREDESTROY: Ibi bikora mbere y'uko component ishira
  // ============================================================
  beforeDestroy() {
    this.stopTimer()  // Ihagarika timer kugirango itakomeze mugihe idakenerwa
  },

  // ============================================================
  // METHODS: Izi ni method (ibikorwa) by'umukino
  // ============================================================
  methods: {
    // ========================================================
    // AMAJWI (SOUND EFFECTS)
    // ========================================================
    preloadAudio() {
      // Iyi method itegura amajwi yose akoreshwa mukino
      this.soundUrls = {
        move: 'https://assets.mixkit.co/sfx/preview/mixkit-select-click-1109.mp3',        // Iyo ukanditse tile
        win: 'https://assets.mixkit.co/sfx/preview/mixkit-correct-answer-tone-2870.mp3',   // Iyo urangiye level
        click: 'https://assets.mixkit.co/sfx/preview/mixkit-software-interface-start-2574.mp3', // Iyo ukanditse buto
        shuffle: 'https://assets.mixkit.co/sfx/preview/mixkit-game-level-complete-2059.mp3',    // Iyo uhinduye tiles
        error: 'https://assets.mixkit.co/sfx/preview/mixkit-wrong-answer-fail-2625.mp3',        // Iyo ukanditse nabi
        levelup: 'https://assets.mixkit.co/sfx/preview/mixkit-unlock-game-notification-2532.mp3', // Iyo ujya muri level nshya
        complete: 'https://assets.mixkit.co/sfx/preview/mixkit-winning-chimes-2015.mp3'         // Iyo urangiye game yose
      }
    },
    
    async playSound(soundType) {
      // Iyi method icuranga amajwi (sound effects)
      // Niba soundEnabled ari false, ntakintu kiba
      if (!this.soundEnabled) return
      
      try {
        const url = this.soundUrls[soundType]
        if (!url) return
        
        const audio = new Audio(url)
        audio.volume = 0.3  // Amajwi agera kuri 30% (ntakabije)
        audio.play().catch(err => {
          // Niba amajwi atapfushije, game ikomeje (nta bugorane)
          console.log('Audio playback failed:', err.message)
        })
      } catch (error) {
        // Nta bugorane buje, game ikomeje
        console.log('Sound error:', error)
      }
    },
    
    // ========================================================
    // GUTANGIRA NO KUBIKA (START & RESET)
    // ========================================================
    startGame() {
      // Iyi method itangiza umukino (itangira kuri level 1)
      this.gameStarted = true
      this.resetGameState()
      this.startTimer()
      this.playSound('click')
    },
    
    resetGameState() {
      // Iyi method isubiza umukino mu mimerere y'ibanze (score=0, level=1,...)
      this.score = 0
      this.currentLevel = 0
      this.moveCount = 0
      this.totalElapsedSeconds = 0
      this.won = false
      this.gameCompleted = false
      this.tiles = [1, 2, 3, 4, 5, 6, 7, 8, 0]
      this.currentLevelFact = this.facts[0]
      this.shuffleTiles()
    },
    
    resetGame() {
      // Iyi method isubiza umukino rwose (iherezo)
      this.stopTimer()
      this.resetGameState()
      this.startTimer()
      this.playSound('click')
    },
    
    // ========================================================
    // TIMER (IGIHE)
    // ========================================================
    startTimer() {
      // Tangiza timer ya level (itangira kubara amasegonda)
      this.stopTimer()
      this.elapsedSeconds = 0
      this.timer = setInterval(() => {
        if (!this.won && this.gameStarted && !this.gameCompleted) {
          this.elapsedSeconds++
        }
      }, 1000)
    },
    
    stopTimer() {
      // Ihagarika timer (iyo level irangiye cyangwa game ihagaritswe)
      if (this.timer) {
        clearInterval(this.timer)
        this.timer = null
      }
    },
    
    // ========================================================
    // PUZZLE LOGIC (LOGIKA Y'I KINYABUPFURO)
    // ========================================================
    shuffleTiles() {
      // Iyi method ihuza tiles (kubihinduranya) kugirango puzzle itangire
      this.won = false
      this.lastMoveIndex = null
      
      // Fisher-Yates algorithm yo guhinduranya tiles
      do {
        for (let i = this.tiles.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1))
          ;[this.tiles[i], this.tiles[j]] = [this.tiles[j], this.tiles[i]]
        }
      } while (!this.isSolvable())  // Ihindura kugeza puzzle ishoboka (solvable)
      
      this.moveCount = 0
      this.playSound('shuffle')
    },
    
    isSolvable() {
      // Iyi method ireba niba puzzle ishoboka (solvable)
      // Kubara inversion count - kuri 3x3, inversion count igomba kuba even
      let inversionCount = 0
      for (let i = 0; i < this.tiles.length - 1; i++) {
        for (let j = i + 1; j < this.tiles.length; j++) {
          if (this.tiles[i] !== 0 && this.tiles[j] !== 0 && this.tiles[i] > this.tiles[j]) {
            inversionCount++
          }
        }
      }
      return inversionCount % 2 === 0
    },
    
    moveTile(index) {
      // Iyi method ikora iyo umukoresh akanditse tile
      if (this.won || this.gameCompleted) return
      
      const emptyIndex = this.tiles.indexOf(0)  // Shaka aho ubusa buri
      
      // Reba niba tile ukanditse iri hafi y'umwanya ubusa
      if (this.isAdjacent(index, emptyIndex)) {
        // Hinduranya tile n'umwanya ubusa
        ;[this.tiles[index], this.tiles[emptyIndex]] = [this.tiles[emptyIndex], this.tiles[index]]
        
        this.moveCount++
        this.lastMoveIndex = index
        setTimeout(() => { this.lastMoveIndex = null }, 200)  // Animation
        this.playSound('move')  // Cura amajwi yo gukanda
        
        this.checkWin()  // Reba niba puzzle yarangiye
      } else {
        this.playSound('error')  // Niba tile itari hafi, curura error sound
      }
    },
    
    isAdjacent(index, emptyIndex) {
      // Iyi method ireba niba indome (index) ebyiri ziri hafi (adjacent)
      const row1 = Math.floor(index / 3)
      const col1 = index % 3
      const row2 = Math.floor(emptyIndex / 3)
      const col2 = emptyIndex % 3
      
      return (Math.abs(row1 - row2) + Math.abs(col1 - col2)) === 1
    },
    
    // ========================================================
    // AMANOTA N'INTSINZI (SCORE & WIN)
    // ========================================================
    calculateLevelScore() {
      // Iyi method ibara amanota wabonye kuri level
      // Formula: 200 + timeBonus + moveBonus
      const timeBonus = Math.max(0, 60 - this.elapsedSeconds) * 5
      const moveBonus = Math.max(0, 100 - this.moveCount) * 3
      
      let levelScore = 200 + timeBonus + moveBonus
      levelScore = Math.max(50, Math.min(1000, levelScore))  // Score hagati ya 50-1000
      
      return Math.floor(levelScore)
    },
    
    checkWin() {
      // Iyi method ireba niba puzzle irangiye (tiles ziri mu buryo bukurikiranwa)
      const correct = [1, 2, 3, 4, 5, 6, 7, 8, 0]  // Ibyo zigomba kuba
      
      if (JSON.stringify(this.tiles) === JSON.stringify(correct)) {
        // Umukoresh yatsinze level!
        this.won = true
        this.lastLevelScore = this.calculateLevelScore()
        this.score += this.lastLevelScore
        this.currentFact = this.facts[this.currentLevel]
        
        // Ongera igihe cya level ku gihe cyose cya game
        this.totalElapsedSeconds += this.elapsedSeconds
        
        this.playSound('win')
        this.stopTimer()
      }
    },
    
    // ========================================================
    // LEVEL ZIKURIKIRA (NEXT LEVEL & COMPLETE)
    // ========================================================
    nextLevel() {
      // Iyi method ijya kuri level ikurikira
      this.won = false
      this.currentLevel++
      this.moveCount = 0
      this.currentLevelFact = this.facts[this.currentLevel]
      this.tiles = [1, 2, 3, 4, 5, 6, 7, 8, 0]
      this.shuffleTiles()
      this.startTimer()
      this.playSound('levelup')
    },
    
    completeGame() {
      // Iyi method irangiza game (nyuma ya level 10)
      this.gameCompleted = true
      this.won = true
      this.stopTimer()
      this.playSound('complete')
    },
    
    // ========================================================
    // IBINDI BIKORWA (TOGGLE SOUND, THEME, ETC)
    // ========================================================
    toggleSound() {
      // Gukoresha / guhagarika amajwi
      this.soundEnabled = !this.soundEnabled
      localStorage.setItem('puzzle-sound', this.soundEnabled)  // Bika preference
      if (this.soundEnabled) {
        this.playSound('click')
      }
    },
    
    toggleTheme() {
      // Guhindura mode (umuco / ijimye)
      this.darkMode = !this.darkMode
      localStorage.setItem('puzzle-theme', this.darkMode ? 'dark' : 'light')
      this.playSound('click')
    },
    
    // ========================================================
    // STYLING Y'IBICE (TILE STYLING)
    // ========================================================
    getTileStyle(tile) {
      // Iyi method igaragariza buri tile amabara n'ifoto igezweho
      const image = this.images[this.currentLevel]  // Ifoto ya level uriho
      
      // Positions za buri tile (x na y) kugirango ifoto igabanijwe neza
      const positions = {
        1: { x: 0, y: 0 },
        2: { x: -100, y: 0 },
        3: { x: -200, y: 0 },
        4: { x: 0, y: -100 },
        5: { x: -100, y: -100 },
        6: { x: -200, y: -100 },
        7: { x: 0, y: -200 },
        8: { x: -100, y: -200 }
      }
      
      return {
        backgroundImage: `url(${image})`,
        backgroundSize: '300px 300px',
        backgroundPosition: `${positions[tile]?.x || 0}px ${positions[tile]?.y || 0}px`,
        backgroundRepeat: 'no-repeat'
      }
    }
  }
}
</script>

<style scoped>
/* ============================================================
   IBIKORWA BYA CSS - STYLING Y'UMUKINO WOSE
   ============================================================ */

/* Base Styles - Imiterere y'ibanze */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.game-container {
  min-height: 100vh;
  transition: all 0.3s ease;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* Theme Styles - Mode y'umuco n'ijimye */
.game-container.dark-theme {
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  color: #eee;
}

.game-container:not(.dark-theme) {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #333;
}

/* ============================================================
   START SCREEN - Screen ibanza
   ============================================================ */
.start-screen {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
}

.start-card {
  max-width: 500px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 40px;
  text-align: center;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  animation: slideUp 0.5s ease;
}

.dark-theme .start-card {
  background: rgba(30, 30, 46, 0.95);
  color: #eee;
}

.start-card h1 {
  font-size: 2rem;
  margin-bottom: 15px;
  color: #667eea;
}

.start-options {
  display: flex;
  gap: 15px;
  justify-content: center;
  margin: 30px 0;
  flex-wrap: wrap;
}

/* Buto zitandukanye */
.start-btn, .theme-btn, .sound-btn {
  padding: 12px 24px;
  border: none;
  border-radius: 25px;
  font-size: 1rem;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.start-btn {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
}

.theme-btn {
  background: #ffd700;
  color: #333;
}

.sound-btn {
  background: #4a5568;
  color: white;
}

/* Hover effects - Iyo umuzi urengaho */
.start-btn:hover, .theme-btn:hover, .sound-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

/* Amabwiriza y'umukino */
.instructions {
  margin-top: 30px;
  text-align: left;
  background: rgba(0, 0, 0, 0.05);
  padding: 20px;
  border-radius: 15px;
}

.instructions h3 {
  margin-bottom: 15px;
  color: #667eea;
}

.instructions p {
  margin: 10px 0;
  font-size: 0.9rem;
}

/* ============================================================
   MAIN GAME CONTAINER
   ============================================================ */
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

/* Game Header - Irumuri rw'umukino */
.game-header h1 {
  font-size: 1.5rem;
  margin-bottom: 20px;
}

@media (min-width: 768px) {
  .game-header h1 {
    font-size: 2rem;
  }
}

/* Stats - Amanota n'ibindi bigaragara neza */
.stats {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  margin-bottom: 20px;
}

.stat-card {
  background: rgba(0, 0, 0, 0.1);
  border-radius: 12px;
  padding: 10px;
  backdrop-filter: blur(10px);
}

.stat-label {
  display: block;
  font-size: 0.75rem;
  opacity: 0.8;
}

.stat-value {
  display: block;
  font-size: 1.25rem;
  font-weight: bold;
}

@media (min-width: 768px) {
  .stat-value {
    font-size: 1.5rem;
  }
}

/* Progress Bar - Intambwe ugezeho */
.progress-section {
  margin-bottom: 20px;
}

.progress-label {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  font-size: 0.85rem;
}

.progress-bar {
  height: 10px;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea, #764ba2);
  border-radius: 10px;
  transition: width 0.3s ease;
}

/* Game Board - Aho puzzle igaragara */
.board-wrapper {
  display: flex;
  justify-content: center;
  margin: 20px 0;
}

.board {
  width: 300px;
  height: 300px;
  display: grid;
  grid-template-columns: repeat(3, 100px);
  grid-template-rows: repeat(3, 100px);
  gap: 0;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  overflow: hidden;
}

/* Tile - Icyiciro cya puzzle */
.tile {
  width: 100%;
  height: 100%;
  border: 1px solid rgba(255, 255, 255, 0.2);
  overflow: hidden;
  cursor: pointer;
  position: relative;
  transition: transform 0.05s linear;
}

.tile:active {
  transform: scale(0.98);
}

/* Animation iyo ukanditse tile */
.move-animation {
  animation: tileMove 0.2s ease;
}

@keyframes tileMove {
  0% { transform: scale(1); }
  50% { transform: scale(0.95); }
  100% { transform: scale(1); }
}

/* Piece - Icyiciro kigizwe n'ifoto */
.piece {
  width: 100%;
  height: 100%;
  background-size: 300px 300px;
  background-repeat: no-repeat;
  position: relative;
}

/* Empty tile - Iyo hari umwanya ubusa */
.empty {
  background: rgba(0, 0, 0, 0.1);
}

/* Number - Numero igaragara kuri buri tile */
.number {
  position: absolute;
  top: 5px;
  left: 5px;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 10px;
  font-weight: bold;
  z-index: 10;
}

/* Buttons z'ubuyobozi */
.buttons {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin: 20px 0;
}

.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 25px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
}

.btn-secondary {
  background: #ffd700;
  color: #333;
  padding: 12px 18px;
}

.btn-success {
  background: linear-gradient(135deg, #11998e, #38ef7d);
  color: white;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

/* Historical Card - Ikarita y'amateka */
.history-card {
  display: flex;
  align-items: center;
  gap: 15px;
  background: rgba(0, 0, 0, 0.1);
  border-radius: 15px;
  padding: 15px;
  margin: 20px 0;
  text-align: left;
  backdrop-filter: blur(10px);
  animation: fadeIn 0.5s ease;
}

.history-icon {
  font-size: 2rem;
}

.history-content h3 {
  font-size: 1rem;
  margin-bottom: 5px;
  color: #ffd700;
}

.history-content p {
  font-size: 0.85rem;
  line-height: 1.4;
}

/* Trophy Screen - Iyo utsinze (Victory Modal) */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
}

.trophy-screen {
  background: white;
  border-radius: 30px;
  padding: 30px;
  max-width: 500px;
  width: 100%;
  text-align: center;
  animation: popIn 0.4s ease;
}

.dark-theme .trophy-screen {
  background: #2d2d44;
  color: white;
}

.trophy-icon {
  font-size: 4rem;
  animation: bounce 0.5s ease;
}

.level-stats {
  margin: 20px 0;
  padding: 15px;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 15px;
}

.fun-fact {
  margin: 20px 0;
  padding: 15px;
  background: linear-gradient(135deg, #667eea20, #764ba220);
  border-radius: 15px;
}

.fun-fact h3 {
  color: #667eea;
  margin-bottom: 10px;
}

.completion-screen {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 2px solid #ffd700;
}

.completion-screen h2 {
  color: #ffd700;
  margin-bottom: 15px;
}

/* ============================================================
   ANIMATIONS
   ============================================================ */
@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes popIn {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-20px);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* ============================================================
   RESPONSIVE DESIGN - Kuri telefone n'ibindi
   ============================================================ */
@media (max-width: 768px) {
  .container {
    padding: 15px;
  }
  
  .stats {
    gap: 8px;
  }
  
  .stat-card {
    padding: 8px;
  }
  
  .btn {
    padding: 10px 20px;
    font-size: 0.9rem;
  }
  
  .trophy-screen {
    padding: 20px;
    margin: 15px;
  }
  
  .history-card {
    padding: 12px;
  }
  
  .history-content p {
    font-size: 0.8rem;
  }
}

@media (max-width: 480px) {
  /* Kuri telefone nto */
  .board {
    width: 270px;
    height: 270px;
    grid-template-columns: repeat(3, 90px);
    grid-template-rows: repeat(3, 90px);
  }
  
  .tile {
    width: 90px;
    height: 90px;
  }
  
  .number {
    font-size: 8px;
    top: 3px;
    left: 3px;
  }
  
  .start-card {
    padding: 25px;
  }
  
  .start-card h1 {
    font-size: 1.5rem;
  }
  
  .instructions p {
    font-size: 0.8rem;
  }
  
  .stats {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .game-header h1 {
    font-size: 1.2rem;
  }
}

/* Landscape Mode - Iyo telefone ihindutse (horizontal) */
@media (max-height: 600px) and (orientation: landscape) {
  .container {
    padding: 10px;
  }
  
  .board {
    width: 240px;
    height: 240px;
    grid-template-columns: repeat(3, 80px);
    grid-template-rows: repeat(3, 80px);
  }
  
  .stats {
    gap: 5px;
  }
  
  .stat-card {
    padding: 5px;
  }
  
  .history-card {
    margin: 10px 0;
  }
}
</style>