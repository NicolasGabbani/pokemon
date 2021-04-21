<template>
  <div class="site">
    <div
      class="modal-content animate__animated animate__faster"
      :class="{ animate__fadeOut: !showPkm, animate__fadeIn: showPkm }"
    >
      <router-view @closeModal="closeModal"></router-view>
    </div>

    <h1 class="logo"><img :src="require('@/assets/images/logo.png')" alt="" /></h1>

    <Loading :loading="loading" />
    <template v-if="!loading">
      <div class="filters">
        <input
          type="text"
          v-model="filter_search"
          placeholder="Recherchez un pokémon..."
        />
        <div class="select">
          <select v-model="filter_type" required>
            <option value="" disabled selected>Filtrer par type...</option>
            <option value="">Tous</option>
            <option value="steel">Acier</option>
            <option value="fighting">Combat</option>
            <option value="dragon">Dragon</option>
            <option value="water">Eau</option>
            <option value="electric">Electrik</option>
            <option value="fairy">Fée</option>
            <option value="fire">Feu</option>
            <option value="ice">Glace</option>
            <option value="bug">Insecte</option>
            <option value="normal">Normal</option>
            <option value="grass">Plante</option>
            <option value="poison">Poison</option>
            <option value="psychic">Psy</option>
            <option value="rock">Roche</option>
            <option value="ground">Sol</option>
            <option value="ghost">Spectre</option>
            <option value="flying">Vol</option>
          </select>
        </div>
      </div>
      <div class="grid pokemons">
        <div class="animate__animated animate__shakeX" v-if="noResult">
          pas de résultat
        </div>
        <div
          v-for="(pokemon, index) in filteredPkms || orderedPkms"
          :key="pokemon.id"
          data-aos="flip-up"
        >
          <router-link
            :to="`/pokemon/${getID(pokemon)}`"
            replace
            class="pokemons__pokemon"
            @mouseenter.native="hoverImg = index"
            @mouseleave.native="hoverImg = null"
          >
            <div class="pokemon-id">#{{ getID(pokemon) }}</div>
            <div class="pokemon-name">{{ getName(pokemon) }}</div>
            <img
              :src="hoverImg == index ? getSpriteShiny(pokemon) : getSprite(pokemon)"
              :alt="getName(pokemon)"
              class="pokemon-img"
            />
            <div class="pokemon-types">
              <span
                class="pokemon-type"
                :class="type"
                v-for="type in pokemon.types"
                :key="type"
              ></span>
            </div>
          </router-link>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import axios from "axios";
import _ from "lodash";
import "animate.css";
import Loading from "@/components/Loading.vue";
import AOS from "aos";
import "aos/dist/aos.css";
export default {
  name: "Home",
  components: {
    Loading,
  },
  data() {
    return {
      pokemons: null,
      arrayPokemons: [],
      filteredPkms: null,
      loading: true,
      hoverImg: null,
      filter_search: "",
      filter_type: "",
      noResult: false,
      showPkm: false,
    };
  },
  mounted() {
    const url = `https://pokeapi.co/api/v2/pokemon?limit=151/`;
    axios
      .get(url)
      .then((response) => (this.pokemons = response.data.results))
      .finally(() => {
        for (let pokemon in this.pokemons) {
          this.getPokemon(this.pokemons[pokemon]);
        }
        AOS.init();
      });
  },
  methods: {
    getPokemon(pkm) {
      let pokemon = null;
      let infoPkm = null;
      axios
        .get(pkm?.url)
        .then((response) => (pokemon = response.data))
        .finally(() => {
          axios
            .get(pokemon?.species.url)
            .then((response) => (infoPkm = response.data))
            .finally(() => {
              this.arrayPokemons.push(infoPkm);
              this.arrayPokemons.find((p) => p.id == pokemon.id).sprite =
                pokemon?.sprites.front_default;
              this.arrayPokemons.find((p) => p.id == pokemon.id).sprite_shiny =
                pokemon?.sprites.front_shiny;
              let typesArr = [];
              for (let index in pokemon?.types) {
                typesArr.push(pokemon?.types[index].type.name);
              }
              this.arrayPokemons.find((p) => p.id == pokemon.id).types = typesArr;
              if (this.arrayPokemons.length == 151) {
                this.loading = false;
              }
            });
        });
    },
    getName(pkm) {
      return pkm?.names.find((name) => name.language.name == "fr").name;
    },
    getSprite(pkm) {
      return pkm.sprite;
    },
    getSpriteShiny(pkm) {
      return pkm.sprite_shiny;
    },
    getID(pkm) {
      return pkm.id;
    },
    closeModal() {
      this.showPkm = false;
      setTimeout(() => {
        this.$router.replace("/");
      }, 500);
    },
    filterName(pkm, val) {
      return val == ""
        ? true
        : this.getName(pkm).toLowerCase().includes(val.toLowerCase()) || pkm.id == val;
    },
    filterType(pkm, val) {
      return val == "" ? true : pkm.types.includes(val);
    },
    filter() {
      if (this.$timer) {
        clearTimeout(this.$timer);
      }
      this.$timer = setTimeout(() => {
        this.filteredPkms = _.orderBy(
          this.arrayPokemons.filter(
            (pkm) =>
              this.filterName(pkm, this.filter_search) &&
              this.filterType(pkm, this.filter_type)
          ),
          "id"
        );
        this.noResult = !this.filteredPkms.length;
      }, 500);
    },
  },
  computed: {
    orderedPkms() {
      return _.orderBy(this.arrayPokemons, "id");
    },
  },
  watch: {
    filter_search: function () {
      this.filter();
    },
    filter_type: function () {
      this.filter();
    },
    $route: {
      immediate: true,
      handler: function (val) {
        this.showPkm = val?.meta.showPkm;
      },
    },
  },
};
</script>
