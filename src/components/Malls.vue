<template>
  <div>
    <NavBar></NavBar>
    <div id="wrapper">
    <h1>Haha</h1>
      <ul id="malls">
        <li class="mall-items" v-for="mall in mallsList" v-bind:key="mall.id">
          <h2>{{ mall.mallName }}</h2>
          <h3>{{ mall.address }}</h3>
          <h3>Singapore {{ mall.postalCode }}</h3>
        </li>
      </ul>
      <Map />
    </div>
  </div>
</template>

<script>
import { database } from "../firebase.js";
import Map from "./MallsMap.vue";

export default {
  name: "Malls",

  data() {
    return {
      mallsList: [],
    };
  },

  components: {
    Map,
  },

  methods: {
    fetchItems: function () {
      database
        .collection("malls")
        .get()
        .then((querySnapShot) => {
          let mall = {};
          querySnapShot.forEach((doc) => {
            mall = doc.data();
            mall.show = false;
            mall.id = doc.id;
            this.mallsList.push(mall);
          });
        });
    },
  },

  created() {
    this.fetchItems();
  },
};
</script>

<style scoped>
h1 {
  font-size: 40px;
  font-weight: normal;
}
#wrapper {
  display: block;
  margin-left: 300px;
  text-align: center;
  grid-template-rows: 50% 50%;
  grid-template-areas:
    "a"
    "b";
  font-family: sans-serif;
}

h3 {
  font-weight: normal;
}

#malls {
  display: flex;
  flex-wrap: wrap;
  list-style-type: none;
  grid-area: a;
  margin-top: 40px;
  margin-bottom: 40px;
}

.mall-items {
  flex-grow: 1;
  flex-basis: 300px;
  text-align: center;
  padding-left: 50px;
  padding-right: 75px;
}

map {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  grid-area: b;
  align-items: center;
  justify-content: center;
}
</style>
