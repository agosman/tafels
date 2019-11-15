<template>
  <div class="table-page">
    <div class="container mx-auto">
      <h2 class="heading">De tafel van {{ table }}</h2>

      <button v-if="!started" @click="onStart">start!</button>
      <div v-else>
        <h3 class="heading">{{ index }} x {{ table }} = ?</h3>
        <input @blur="onInputBlur" @keyup.enter="onEnter" />
      </div>
    </div>
  </div>
</template>

<script>
const INDICES = [...Array(10).keys()];

export default {
  data() {
    console.log('evaluating data. table:', this.table);
    return {
      started: false,
      completed: false,
      index: 1,
      answers: INDICES.map((i, index) => {
        return {
          i,
          answer: null,
          solution: (index + 1) * this.table,
        };
      }),
    };
  },
  computed: {
    table() {
      return Number(this.$route.params.table);
    },
  },
  methods: {
    onStart() {
      this.started = true;
    },
    onInputBlur() {
      console.log('onInputBlur');
    },
    onEnter() {
      console.log('onEnter');
      this.index++;
    },
  },
};
</script>

<style scoped>
input {
  border: 1px solid rgba(0, 0, 0, 0.3);
  padding: 10px 20px;
}
</style>
