<template>
  <div class="modal d-flex animate__animated">
    <Loading :loading="loading" />
    <div
      v-if="!loading"
      class="this-pokemon animate__animated"
      :class="{ animate__fadeIn: !loading }"
    >
      <div class="col">
        <span class="id">{{ pokemon.id }}</span>
        <button class="back" @click.prevent="$emit('closeModal')">X</button>
        <h1>{{ namePkm }}</h1>
        <p class="text-center">{{ descPkm }}</p>
        <p class="types text-center">
          <span class="type" :class="type.en" v-for="type in typesPkm" :key="type.en">{{
            type.fr
          }}</span>
        </p>
      </div>
      <div class="col">
        <img :src="this.getSprite()" :alt="namePkm" />
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Loading from "@/components/Loading.vue";
export default {
  name: "Home",
  components: {
    Loading,
  },
  props: {
    closeModal: Function,
  },
  data() {
    return {
      loading: true,
      pokemon: null,
      namePkm: null,
      descPkm: null,
      typesPkmTmp: [],
      typesPkm: [],
      spritePkm: null,
      id: this.$route.params.id,
    };
  },
  mounted() {
    this.loadPkm();
  },
  methods: {
    loadPkm() {
      this.loading = true;
      const url = `https://pokeapi.co/api/v2/pokemon/${this.id}`;
      axios
        .get(url)
        .then((response) => (this.pokemon = response.data))
        .finally(() => {
          for (let index in this.pokemon?.types) {
            this.typesPkmTmp.push(this.pokemon?.types[index].type);
          }
          this.getInfo();
          this.loading = false;
        });
    },
    async getInfo() {
      let infoPkm = null;
      let tyPkm = null;
      for (let index in this.typesPkmTmp) {
        await axios
          .get(this.typesPkmTmp[index]?.url)
          .then((response) => (tyPkm = response.data))
          .finally(() => {
            this.typesPkm[index] = {
              en: this.typesPkmTmp[index].name,
              fr: tyPkm.names.find((t) => t.language.name == "fr").name,
            };
          });
      }
      await axios
        .get(this.pokemon?.species.url)
        .then((response) => (infoPkm = response.data))
        .finally(() => {
          this.namePkm = infoPkm?.names.find((name) => name.language.name == "fr").name;
          this.descPkm = infoPkm?.flavor_text_entries.find(
            (desc) => desc.language.name == "fr" && desc.version.name == "lets-go-pikachu"
          ).flavor_text;
        });
    },
    getSprite() {
      return this.pokemon?.sprites.other["official-artwork"].front_default;
    },
  },
  watch: {
    $route(to) {
      this.id = to.params.id;
      this.loadPkm();
    },
  },
};
</script>

<style lang="sass">
@media screen and (max-width:375px)
  .modal
    height: 90vh !important
.modal-content
  position: relative
  z-index: 99
.modal
  width: min(90vw, 900px)
  height: 70vh
  position: fixed
  top: 50%
  left: 50%
  z-index: 99
  transform: translate(-50%, -50%)
  background: #F1F1F1
  border: 1px solid #AAA
  padding: 30px
  border-radius: 10px
.this-pokemon
  display: grid
  grid-template-columns: repeat(auto-fill, minmax(130px, 1fr))
  gap: 50px
  img
    display: block
  .id
    position: absolute
    top: 10px
    left: 15px
    font-weight: bold
    font-size: 1.4rem
    &::before
      content: '#'
.col
  display: flex
  flex-direction: column
  justify-content: center
  align-content: center
.types
  margin-top: 30px
  display: flex
  justify-content: center
  align-content: center
  gap: 15px
  width: 100%
  .type
    padding: 6px 12px
    color: white
    border-radius: 10px

.back
  position: absolute
  top: 10px
  right: 10px
  text-decoration: none
  text-transform: uppercase
  color: black
  font-weight: bold
  border: none
  outline: none
  font-size: 2rem
  cursor: pointer
</style>
