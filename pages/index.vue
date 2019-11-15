<template>
  <div class="container">
    <h2 class="heading">
      De tafels
    </h2>
    <!-- selector -->
    <ol class="table-list">
      <li v-for="i in 10" :key="i" class="table-item">
        <nuxt-link :to="`/${i}`">{{ i }}</nuxt-link>
      </li>
    </ol>
  </div>
</template>

<script>
export default {
  components: {},
};
</script>

<style>
/* Sample `apply` at-rules with Tailwind CSS
.container {
  @apply min-h-screen flex justify-center items-center text-center mx-auto;
}
*/

.table-list {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  grid-column-gap: 10px;
  grid-row-gap: 10px;
}

.table-item {
  @apply text-center;
  width: 44px;
  height: 44px;
  outline: 1px solid rgba(0, 0, 0, 0.3);
  display: flex;
  flex-direction: column;
  justify-content: center;
}
</style>
