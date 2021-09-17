<template>
  <div>
    <Rating :clearable="true" :show-rating="false" @rating-selected="setRating" />
  </div>
</template>

<script>
import Rating from "vue-star-rating";

export default {
  data() {
    return {
      rating: 0,
    };
  },
  components: {
    Rating,
  },
  props: {
    restaurantId: String,
  },
  methods: {
    setRating(rating) {
      this.rating = rating;
      this.$emit("updateRating", rating, this.restaurantId);
    },
  },
};
</script>

<style>
</style>