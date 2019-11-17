<template>
  <div class="page">
    <div class="container mx-auto">
      <h1 class="heading">Hallo {{ value }}</h1>
      <input v-model="value" />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      value: 'Juliette',
    };
  },
};
</script>

<style lang="scss" scoped>
.page {
  height: 100vh;
  overflow: hidden;
  background: linear-gradient(pink, purple);
}

input {
  font-size: 20px;
  color: pink;
  padding: 10px 20px;
  background-color: rgba(0, 0, 0, 0.5);
  text-align: right;
}
</style>
