<template>
  <div class="gameboard" id="App">
    <audio ref="sound1" src="./assets/sounds/1.mp3"></audio>
    <audio ref="sound2" src="./assets/sounds/2.mp3"></audio>
    <audio ref="sound3" src="./assets/sounds/3.mp3"></audio>
    <audio ref="sound4" src="./assets/sounds/4.mp3"></audio>
    <div class="gameboard__box">
      <h1 class="gameboard__title">Simon The Game</h1>
      <div class="gameboard__buttons">
        <button class="button button_blue" :class="{ active : hlBlue }" @click="input(1)"></button>
        <button class="button button_red" :class="{ active : hlRed }" @click="input(2)"></button>
        <button class="button button_green" :class="{ active : hlGreen }" @click="input(3)"></button>
        <button class="button button_yellow" :class="{ active : hlYellow }" @click="input(4)"></button>
      </div>
    </div>
    <div class="sidebar">
      <p class="best-score">Лучший счёт: {{ score }}</p>
      <div class="play">
        <h2 class="rounds"> Раунд: {{ round }}</h2>
        <button class="start" @click="start">Старт</button>
        <p v-if="loss" class="notification">Извините, вы проиграли в {{ round }} раунде!</p>
      </div>
      <div class="difficulty">
        <h2 class="difficulty__title">Cложность</h2>
        <ul class="difficulty__buttons">
          <li class="difficulty__button">
              <input type="radio" value="easy" id="difficulty-easy" name="difficulty" @click="delay = 1500" :disabled="started">
              <label for="difficulty-easy">Легкая</label>
          </li>
          <li class="difficulty__button">
              <input type="radio" value="medium" id="difficulty-medium" name="difficulty" @click="delay = 1000" :disabled="started" checked>
              <label for="difficulty-medium">Средняя</label>
          </li>
          <li class="difficulty__button">
              <input type="radio" value="hard" id="difficulty-hard" name="difficulty" @click="delay = 600" :disabled="started">
              <label for="difficulty-hard">Сложная</label>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      count: 0,
      round: 0,
      started: false,
      series: [],
      userTone: [],
      delay: 1000,
      currentDelay: 0,
      isPlayed: false,
      loss: false,
      score: 0,

      hlBlue: false,
      hlGreen: false,
      hlYellow: false,
      hlRed: false
    }
  },

  created: function() {
    this.bestScore();
  },

  methods: {
    start() {
      this.currentDelay = this.delay;
      this.round = 0;
      this.started = true;
      this.loss = false;
      this.series = [];
      this.userTone = [];
      this.addTone();
      this.playSeries();
    },

    input(tone) {
      if (this.started && !this.isPlayed) {
        this.playTone(tone);
        setTimeout(this.clearHighlights, 300);

        if (tone === this.series[this.count]) {
          this.userTone.push(tone);
          this.count++;

          if (this.series.length == this.userTone.length) {
            this.userTone = [],
            this.addTone();
            this.count = 0;
            this.currentDelay = this.delay;
            this.playSeries();
          }

        } else {
          this.setBestScore();
          this.reset();
          this.loss = true;
        }
      }
    },

    clearHighlights() {
      this.hlGreen = false;
      this.hlRed = false;
      this.hlYellow = false;
      this.hlBlue = false;
    },

    playTone(tone) {
      switch(tone) {
        case 1: 
          this.$refs.sound1.play();
          this.hlBlue = true;
          break;
        case 2: 
          this.$refs.sound2.play();
          this.hlRed = true;
          break;
        case 3: 
          this.$refs.sound3.play();
          this.hlGreen = true;
          break;
        case 4: 
          this.$refs.sound4.play();
          this.hlYellow = true;
          break;
      }
    },

    reset() {
      this.count = 0;
      this.started = false;
      this.series = [];
      this.userTone = [];
      this.currentDelay = 0;
    },

    addTone() {
      this.series.push(this.randomTone());
      this.round++;
    },
    
    randomTone() {
      return Math.floor(Math.random() * 4) + 1;
    },

    playSeries() {
      this.isPlayed = true;

      this.series.forEach(item => {
        setTimeout(() => {
          this.playTone(item);
          setTimeout(this.clearHighlights, 300);
        }, this.currentDelay);
        this.currentDelay += this.delay;  
      });

      setTimeout(() => {
        this.isPlayed = false;
      }, this.delay * this.series.length);
    },

    bestScore() {
      if (localStorage.getItem('score')) {
        this.score = localStorage.getItem('score');
      } else {
        localStorage.setItem('score', 0);
      }
    },  

    setBestScore() {
      if (this.round - 1 > localStorage.getItem('score')) {
        localStorage.setItem('score', this.round - 1);
        this.score = localStorage.getItem('score');
      }
    }
  }
}
</script>

<style lang="scss">

   h1, h2 {
     letter-spacing: 1.5px;
   }  

    @import url('https://fonts.googleapis.com/css2?family=Underdog&display=swap');

    .gameboard {
      display: flex;
      min-height: 100vh;
      align-items: center;
      justify-content: center;
      font-family: 'Underdog', cursive;
      margin-top: -30px;
    }

    .gameboard__box {
      max-width: 500px;
      width: 100%;
      margin-right: 50px;
    }

    .gameboard__title {
      margin: 0 0 30px;
      text-align: center;
    }

   .gameboard__buttons {
     display: flex;
     max-width: 500px;
     width: 100%;
     flex-wrap: wrap;
     border: 3px solid #fff;
     border-radius: 250px;
     box-shadow: 0 2.5px 10px 0 rgba(0,0,0,.5);
   }

   .difficulty__buttons {
     padding: 0;
     margin: 0;
     list-style: none;
   }

   .sidebar {
     width: 250px;
   }

   .difficulty__button {

     & input {
       margin-right: 10px;
     }

     & label {
       cursor: pointer;
     }
   }

   .button {
     width: 50%;
     height: 250px;
     border: none;
     opacity: 0.6;
     outline: none;
     border: 5px solid #fff;
     cursor: pointer;

     &_blue {
       background-color:dodgerblue;
       border-radius: 250px 0 0 0;
     }

     &_red {
       background-color: #FF5643;
       border-radius: 0 250px 0 0;
     }

     &_green {
       background-color:#BEDE15;
       border-radius: 0 0 0 250px;
     }

     &_yellow {
       background-color:#FEEF33;
       border-radius: 0 0 250px 0;
     }
   }

   .active {
     opacity: 1;
   }

   .start {
     font: inherit;
     padding: 10px 45px;
     cursor: pointer;
     background-color:dodgerblue;
     color: #fff;
     border: none;

     &:hover {
       opacity: 0.8;
    }
   }

  .best-score {
    color: #FF5643;
    font-weight: 700;
  }
</style>>

 

