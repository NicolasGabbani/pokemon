<template>
  <div v-if="loading" class="loading">
    <div class="pokeball-content"><div class="pokeball"></div></div>
  </div>
</template>

<script>
export default {
  name: 'Loading',
  props: {
    loading: Boolean
  }
}
</script>