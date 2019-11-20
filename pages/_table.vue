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
          <h3>Je hebt er {{ score }} goed.</h3>
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
  </div>
</template>

<script>
import gsap from 'gsap';

const NUM_QUESTIONS = 10;

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
    console.log('* updated *');
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
        this.completed = true;
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
      // this.currentItem.correct = this.currentItem.answer === this.currentItem.solution;
    },
    onStart() {
      this.started = true;
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

      // clear answers and correct properties
      this.initializeItems();
    },
  },
};
</script>

<style lang="scss" scoped>
input {
  border: 1px solid rgba(0, 0, 0, 0.3);
  padding: 10px 20px;
  font-size: 26px;
  color: #000;
  text-align: center;
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

.restart-btn {
  margin: 20px 0;
}
</style>
