<template>
  <div class="table-page">
    <div class="container mx-auto">
      <h2 class="title heading">De tafel van {{ table }}</h2>

      <button v-if="!started" @click="onStart" class="btn">start!</button>
      <div v-else class="questions">
        <div ref="question" v-if="!completed" :class="{ correct: currentItem.correct === true }" class="question">
          <h3 class="heading">{{ currentItem.multiplier }} x {{ table }} = ?</h3>
          <input
            ref="input"
            v-model.number="currentItem.answer"
            @blur="onInputBlur"
            @keyup.enter="onEnter"
            :maxlength="maxLength"
            type="number"
          />
        </div>
        <div v-else>
          <h2 class="heading">Je bent klaar!</h2>
          <h3>
            Je hebt er <span class="score">{{ score }}</span> goed.
          </h3>
          <h4>Tijd: {{ formattedTimeSpent }}</h4>
          <button @click="onReset" class="btn restart-btn">Opnieuw beginnen</button>
        </div>
        <div class="progress">
          <div
            v-for="(item, index) in items"
            :key="`progress-${item.multiplier}`"
            :class="{ current: index === currentIndex, correct: item.correct, incorrect: item.correct === false }"
            class="progress-item"
          />
        </div>
      </div>
    </div>
    <div class="progress-bars">
      <div :style="progressBarTimeStyle" class="progress-bar progress-bar-time" />
      <div :style="progressBarQuestionsStyle" class="progress-bar progress-bar-questions" />
    </div>
  </div>
</template>

<script>
import gsap from 'gsap';

const NUM_QUESTIONS = 10;
const MAX_TIME = 60; // in seconds

const INDICES = [...Array(NUM_QUESTIONS).keys()];

export default {
  data() {
    console.log('evaluating data. table:', this.table);
    return {
      started: false,
      completed: false,
      currentIndex: 0,
      maxLength: 3,
      answer: '',
      items: null,
      startTime: null,
      timeSpent: 0,
    };
  },
  computed: {
    table() {
      return Number(this.$route.params.table);
    },
    status() {
      if (!this.answer.length) {
        return 'Typ maar eens even wat hoor...';
      }

      if (this.answer.length >= this.maxLength) {
        return 'Je bent wel klaar nu hoor...';
      }

      return `Je hebt ${this.answer.length || 0} karakters getypt.`;
    },
    currentItem() {
      return this.items[this.currentIndex];
    },
    // done() {
    //   return this.currentIndex > 0 && this.currentIndex >= this.items.length;
    // },
    score() {
      return this.items.filter((item) => item.correct === true).length;
    },
    formattedTimeSpent() {
      const s = Math.round(this.timeSpent / 100) / 10;
      return `${s} seconden`;
    },
    progress() {
      if (!this.started) {
        return 0;
      }
      return (this.currentIndex + 1) / NUM_QUESTIONS;
    },
    timeProgress() {
      return Math.min(1, this.timeSpent / 1000 / MAX_TIME);
    },
    progressBarTimeStyle() {
      return {
        transform: `scaleX(${this.timeProgress})`,
      };
    },
    progressBarQuestionsStyle() {
      return {
        transform: `scaleX(${this.progress})`,
      };
    },
  },
  watch: {
    currentIndex(value) {
      gsap.to(this.$refs.question, { duration: 0.3, opacity: 1 });
    },
  },
  mounted() {
    this.initializeItems();
  },
  updated() {
    if (!this.completed && this.started) {
      this.$refs.input.focus();
    }
  },
  methods: {
    initializeItems() {
      this.items = INDICES.map((i) => {
        const multiplier = i + 1;
        return {
          random: Math.random(),
          multiplier,
          answer: null,
          solution: multiplier * this.table,
        };
      }).sort((a, b) => {
        return a.random < b.random ? -1 : 1;
      });
    },
    proceed() {
      this.$refs.input.blur();

      this.validate();

      if (this.currentIndex < NUM_QUESTIONS - 1) {
        this.next();
      } else {
        this.finish();
      }
    },
    next() {
      gsap.to(this.$refs.question, {
        duration: 0.3,
        opacity: 0,
        onComplete: () => {
          this.currentIndex++;
        },
      });
    },
    validate() {
      const newValue = {
        ...this.currentItem,
        correct: this.currentItem.answer === this.currentItem.solution,
      };
      this.$set(this.items, this.currentIndex, newValue);
    },
    trackTime() {
      this.timeSpent = Date.now() - this.startTime;
    },
    finish() {
      this.timeSpent = Date.now() - this.startTime;
      this.completed = true;
      clearInterval(this.trackTimeInterval);
    },
    onStart() {
      this.startTime = Date.now();
      this.started = true;
      this.trackTimeInterval = setInterval(() => {
        this.trackTime();
      }, 50);
    },
    onInputBlur() {
      console.log('onInputBlur');
    },
    onEnter() {
      console.log('onEnter');
      this.proceed();
    },
    onQuestionEnter(el, done) {
      console.log('onQuestionEnter');
      gsap.fromTo(el, { opacity: 0 }, { duration: 1, opacity: 1, onComplete: done });
      // done();
    },
    onQuestionLeave(el, done) {
      console.log('onQuestionLeave');
      gsap.to(el, { duration: 1, opacity: 1, onComplete: done });
      // done();
    },
    onReset() {
      this.started = false;
      this.completed = false;
      this.currentIndex = 0;
      this.timeSpent = 0;

      // clear answers and correct properties
      this.initializeItems();
    },
  },
};
</script>

<style lang="scss" scoped>
.table-page {
  height: 100%;
}

input {
  border: 1px solid rgba(0, 0, 0, 0.3);
  padding: 10px 20px;
  font-size: 26px;
  color: #000;
  text-align: center;

  &:focus {
    outline: 0;
  }
}

.title {
  font-size: 48px;
  padding: 0;
  margin-bottom: 20px;
  border-bottom: 1px solid rgba(255, 225, 255, 0.3);
}

.progress {
  display: flex;
  justify-content: center;
  margin: 40px 0;
}

.progress-item {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.4);
  margin: 0 8px;

  &.current {
    background-color: #fff;
  }

  &.correct {
    background-color: #7cff7c;
  }

  &.incorrect {
    background-color: #ff0000;
  }
}

.progress-bars {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 8px;
  background-color: rgba(#000, 0.4);
}

.progress-bar {
  height: 50%;
  transform: scaleX(0);
  transform-origin: left;
}

.progress-bar-time {
  background-color: rgba(#fff, 0.6);
}

.progress-bar-questions {
  transition: transform 0.5s;
  background-color: #ac1d72;
}

.restart-btn {
  margin: 20px 0;
}
</style>
