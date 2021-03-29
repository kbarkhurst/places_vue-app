<template>
  <div class="home">
    <h1>{{ message }}</h1>
    <h2>Place</h2>
    <p>Add a new Place</p>

    <ul
      v-if="errors.length"
      style="
        color: red;
        list-style: none;
        text-align: center;
        border: solid 1px red;
        width: 70%;
        padding: 4%;
        margin-left: auto;
        margin-right: auto;
      "
    >
      <li v-for="error in errors" v-bind:key="error">WARNING: {{ error }}</li>
    </ul>

    <input type="text" v-model="newPlaceName" />
    <p>Address</p>
    <input type="text" v-model="newPlaceAddress" />
    <button v-on:click="createPlace()">Create New Place</button>

    <div v-for="place in places" v-bind:key="place.id" style="border-top: 1px solid #ccc; margin: 2% auto">
      <h2>{{ place.name }}</h2>
      <p>{{ place.address }}</p>
      {{ place.id }}
      <button v-on:click="showPlace(place)">More Info!</button>
    </div>
    <dialog id="place-details">
      <form method="dialog">
        <h1>Place Info</h1>
        <p>
          Name:
          <input type="text" v-model="currentPlace.name" />
        </p>
        <p>
          Address:
          <input type="text" v-model="currentPlace.address" />
        </p>

        <button v-on:click="destroyPlace(currentPlace)">Delete</button>
        <button v-on:click="updatePlace(currentPlace)">Update</button>

        <button>Close</button>
      </form>
    </dialog>
  </div>
</template>

<style></style>

<script>
import axios from "axios";
export default {
  data: function () {
    return {
      message: "Welcome. Explore these Places shown by JS.",
      places: [],
      newPlaceName: "",
      newPlaceAddress: "",
      currentPlace: {},
      errors: [],
    };
  },
  created: function () {
    this.indexPlaces();
  },
  methods: {
    indexPlaces: function () {
      axios.get("/api/places").then((response) => {
        this.places = response.data;
        console.log("all places:", this.places);
      });
    },
    createPlace: function () {
      console.log("Creating a place!");
      var params = {
        name: this.newPlaceName,
        address: this.newPlaceAddress,
      };
      axios
        .post("/api/places", params)
        .then((response) => {
          console.log(response.data);
          this.places.push(response.data);
        })
        .catch((error) => {
          this.errors = error.response.data.errors;
        });
    },
    showPlace: function (place) {
      console.log(place);
      this.currentPlace = place;
      document.querySelector("#place-details").showModal();
    },
    updatePlace: function (place) {
      var params = {
        name: place.name,
        address: place.address,
      };
      axios
        .patch("/api/places/" + place.id, params)
        .then((response) => {
          console.log("Updated Successfully", response.data);
        })
        .catch((error) => {
          this.errors = error.response.data.errors;
        });
    },
    destroyPlace: function (place) {
      console.log("Deleting...");
      axios.delete("/api/places/" + place.id).then((response) => {
        console.log("Success!", response.data);
        var index = this.places.indexOf(place);
        this.places.splice(index, 1);
      });
    },
  },
};
</script>
