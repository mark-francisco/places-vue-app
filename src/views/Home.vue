<template>
  <div class="home">
    <h1>{{ title }}</h1>
    <h2>Create a New Place!</h2>
    <p>
      <label for="name">Name:</label>
      <input v-model="newPlaceName" id="name" type="text" />
    </p>
    <p>
      <label for="address">Address:</label>
      <input v-model="newPlaceAddress" id="address" type="text" />
    </p>
    <button v-on:click="createPlace()">Create New Place!</button>
    <ul id="error-list">
      <li v-for="error in errors" v-bind:key="error">{{ error }}</li>
    </ul>

    <div v-for="place in places" v-bind:key="place.id">
      <hr />
      <p>{{ place.name }}</p>
      <button v-on:click="showPlace(place)">More Info:</button>
      <hr />
    </div>
    <dialog id="place-details">
      <form method="dialog">
        <p>ID: {{ currentPlace.id }}</p>
        <p>
          Name:
          <input v-model="currentPlace.name" type="text" />
        </p>
        <p>
          Address:
          <input v-model="currentPlace.address" type="text" />
        </p>
        <button v-on:click="updatePlace(currentPlace)">UPDATE:</button>
        <button v-on:click="destroyPlace(currentPlace)">DESTROY:</button>
        <button>close:</button>
      </form>
    </dialog>
  </div>
</template>

<style>
#error-list {
  color: red;
  font-weight: bold;
}
</style>

<script>
import axios from "axios";

export default {
  data: function () {
    return {
      title: "Places Vue app",
      places: [],
      newPlaceName: "",
      newPlaceAddress: "",
      currentPlace: "",
      errors: [],
    };
  },
  created: function () {
    this.indexPlaces();
  },
  methods: {
    indexPlaces: function () {
      axios.get("/api/places").then((res) => {
        this.places = res.data;
        console.log("INDEX Places", res.data);
      });
    },
    createPlace: function () {
      let params = {
        name: this.newPlaceName,
        address: this.newPlaceAddress,
      };
      axios
        .post("/api/places", params)
        .then((res) => {
          console.log(res.data);
          this.places.push(res.data);
          this.newPlaceName = "";
          this.newPlaceAddress = "";
          this.errors = [];
        })
        .catch((err) => {
          this.errors = err.response.data.errors;
          console.log("ERRORS with CREATE:", err.response.data.errors);
        });
    },
    showPlace: function (place) {
      this.currentPlace = place;
      document.querySelector("#place-details").showModal();
    },
    updatePlace: function (place) {
      let params = {
        name: place.name,
        address: place.address,
      };
      axios
        .patch(`/api/places/${place.id}`, params)
        .then((res) => {
          console.log("PLACE UPDATED:", res.data);
          this.errors = [];
        })
        .catch((err) => {
          this.errors = err.response.data.errors;
          console.log("ERRORS with UPDATE:", err.response);
        });
    },
    destroyPlace: function (place) {
      axios.delete("/api/places/" + place.id).then((res) => {
        console.log("PLACE DELETED:", res.data);
        let index = this.places.indexOf(place);
        this.places.splice(index, 1);
      });
    },
  },
};
</script>
