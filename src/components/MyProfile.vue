<template>
  <div>
    <NavBar></NavBar>
    <div id="wrapper">
      <div
        id="profile"
        v-if="this.editProfile == false && this.changePassword == false"
      >
        <div id="profileIcon">
          <profileIcon :size="200" />
        </div>
        <div id="details">
          <p class="name">{{ this.name }}</p>
          <p class="details">Email Address: {{ this.email }}</p>
          <p class="details">Contact Number: {{ this.contact }}</p>
          <p class="details">Date of Birth: {{ this.dob }}</p>
          <button v-on:click="edit()">Edit Profile</button>
          <button v-on:click="change()">Change Password</button>
        </div>
      </div>

      <div
        id="editProfile"
        v-if="this.editProfile == true && this.changePassword == false"
      >
        <div id="profileIcon">
          <profileIcon :size="200" />
        </div>
        <div id="editDetails">
          <form @submit="edit()">
            <p class="editDetails">
              <input
                input
                type="text"
                name="name"
                id="name"
                v-model="name"
                placeholder="Name"
                required
              />
            </p>
            <p class="editDetails">
              <input
                type="tel"
                name="contact"
                id="contact"
                v-model="contact"
                placeholder="Contact Number"
                pattern="[7-9]{1}[0-9]{7}"
                required
              />
            </p>
            <p class="editDetails">
              <input
                type="date"
                name="dob"
                id="dob"
                v-model="dob"
                placeholder="Date of Birth"
                required
              />
            </p>
            <button type="submit">Save</button>
            <button type="button" v-on:click="cancel()">Cancel</button>
          </form>
        </div>
      </div>

      <div
        id="changePassword"
        v-if="this.changePassword == true && this.editProfile == false"
      >
        <div id="profileIcon">
          <profileIcon :size="200" />
        </div>
        <div id="editPassword">
          <p class="editPassword">
            <input
              type="password"
              name="oldPassword"
              id="oldPassword"
              v-model="oldPassword"
              placeholder="Old Password"
              required
            />
          </p>
          <p class="editPassword">
            <input
              type="password"
              name="newPassword"
              id="newPassword"
              v-model="newPassword"
              placeholder="New Password"
              required
            />
          </p>
          <p class="editPassword">
            <input
              type="password"
              name="confirmPassword"
              id="confirmPassword"
              v-model="confirmPassword"
              placeholder="Confirm New Password"
              required
            />
          </p>
          <button v-on:click="change()">Save</button>
          <button v-on:click="cancel()">Cancel</button>
        </div>
      </div>

      <h2>
        Current Queue Details
      </h2>
      <div id="currentQueue">
        <div id="currentQueueDetails" v-if="this.waitingRestaurant != ''">
          <p>
            Please arrive at {{ this.waitingRestaurant }} @
            {{ this.waitingMall }} by {{ this.arrivalTime }}.
          </p>
          <p>Details:</p>
          <span>Number of Pax (Adult): {{ this.numOfAdult }}</span
          ><br />
          <span>Number of Pax (Children): {{ this.numOfChildren }}</span
          ><br />
          <span>Number of Baby Chair(s): {{ this.babychair }}</span
          ><br />
          <span>Number of Wheelchair(s): {{ this.wheelchair }} </span><br />
        </div>
        <div id="currentQueueDetails" v-if="this.waitingRestaurant == ''">
          <p class="notInQ">
            Currently not in any queue. Click on 'Join a Queue' to join a queue.
          </p>
        </div>
      </div>
      <p class="note">
        Note: Full party must be present to be seated. <br />
        The queue seatings will be given up if you are late due to limited
        seating capacity.
      </p>

      <h2>
        Queue History
      </h2>
      <div id="queueHistory">
        <div id="queueHistoryDetails">
          <ul>
            <li
              v-for="booking in history"
              v-bind:key="booking.id"
              class="queueHistory"
            >
              <span class="historyRestaurantMall">{{
                booking.restaurantMall
              }}</span>
              <rating
                @updateRating="ratingUpdated"
                v-bind:restaurantId="booking.restaurantId"
              ></rating>
              <button class="rate" v-on:click="rate()">Rate</button>
            </li>
            <br />
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { database } from "../firebase.js";
import { auth } from "../firebase.js";
import firebase from "firebase/app";
import profileIcon from "vue-material-design-icons/AccountCircle";
import rating from "./Rating.vue";

export default {
  data() {
    return {
      userid: "",
      editProfile: false,
      changePassword: false,
      name: "",
      email: "",
      contact: null,
      dob: "",
      oldPassword: null,
      newPassword: null,
      confirmPassword: null,
      numOfAdult: 0,
      numOfChildren: 0,
      babychair: 0,
      wheelchair: 0,
      waitingRestaurant: "",
      waitingMall: "",
      arrivalTime: "",
      history: [],
      rating: 0,
      rateRestId: "",
      rateTtlRatings: 0,
      rateNumRatings: 0,
    };
  },
  methods: {
    fetchItems() {
      database
        .collection("users")
        .doc(`${auth.currentUser.uid}`)
        .get()
        .then((querySnapShot) => {
          this.name = querySnapShot.data().name;
          this.email = querySnapShot.data().email;
          this.contact = querySnapShot.data().contact;
          this.dob = querySnapShot.data().dob;
          this.userid = querySnapShot.data().customerID;
        });
    },
    edit() {
      if (this.editProfile == false) {
        this.editProfile = true;
      } else {
        database
          .collection("users")
          .doc(`${auth.currentUser.uid}`)
          .update({
            name: this.name,
            contact: this.contact,
            dob: this.dob,
          });
        this.editProfile = false;
        alert("Profile successfully updated.");
      }
    },
    reauthenticate(password) {
      const user = auth.currentUser;
      const cred = firebase.auth.EmailAuthProvider.credential(
        user.email,
        password
      );
      return user.reauthenticateWithCredential(cred);
    },
    change() {
      if (this.changePassword == false) {
        this.changePassword = true;
      } else {
        if (
          this.oldPassword == null ||
          this.newPassword == null ||
          this.confirmPassword == null
        ) {
          alert("Please fill out all the fields and try again.");
        } else {
          this.reauthenticate(this.oldPassword)
            .then(() => {
              if (this.newPassword.length < 6) {
                alert(
                  "Password must contain at least 6 characters. Please check your password and try again."
                );
              } else if (this.newPassword != this.confirmPassword) {
                alert(
                  "Passwords do not match. Please check your password and try again."
                );
              } else {
                auth.currentUser.updatePassword(this.newPassword).then(() => {
                  alert("Password successfully changed.");
                  this.changePassword = false;
                  this.editProfile = false;
                });
              }
            })
            .catch((error) => {
              if (error.code == "auth/wrong-password") {
                alert(
                  "The current password you entered is wrong. Please check your current password and try again."
                );
              }
            });
        }
      }
    },
    cancel() {
      this.editProfile = false;
      this.changePassword = false;
      this.fetchItems();
    },
    getCurrentAndHist() {
      database
        .collection("bookings")
        .get()
        .then((snapshot) => {
          snapshot.docs.forEach((doc) => {
            var queue = doc.data();
            if (
              this.userid === queue.customerID &&
              queue.queueStatus === "waiting"
            ) {
              this.numOfAdult = queue.numAdult;
              this.numOfChildren = queue.numChildren;
              this.babychair = queue.babyChair;
              this.wheelchair = queue.wheelChair;
              this.waitingMall = queue.mallName;
              this.waitingRestaurant = queue.restaurantName;
              this.arrivalTime = queue.arrivalTime;
            } else if (
              this.userid === queue.customerID &&
              queue.queueStatus === "completed"
            ) {
              this.history.push(queue);
            }
          });
        });
    },
    ratingUpdated(rating, restId) {
      this.rating = rating;
      this.rateRestId = restId;
    },
    rate() {
      database
        .collection("restaurants")
        .get()
        .then((snapshot) => {
          snapshot.docs.forEach((doc) => {
            if (doc.id === this.rateRestId) {
              this.rateTtlRatings = doc.data().totalRatings;
              this.rateNumRatings = doc.data().numRatings;
            }
          });
        });
      database
        .collection("restaurants")
        .doc(this.rateRestId)
        .update({
          rating:
            (this.rating + this.rateTtlRatings) / (this.rateNumRatings + 1),
          totalRatings: this.rating + this.rateTtlRatings,
          numRatings: this.rateNumRatings + 1,
        });
      alert("Restaurant successfully rated.");
    },
  },
  created() {
    this.fetchItems();
    this.getCurrentAndHist();
  },
  components: {
    profileIcon,
    rating,
  },
};
</script>

<style scoped>
#wrapper {
  margin-left: 300px;
  font-family: sans-serif;
}
#profileIcon {
  float: left;
  padding-left: 10px;
  padding-top: 45px;
}
#profile {
  border: 1px solid #222;
  background-color: white;
  height: 300px;
  width: 800px;
  border-radius: 10px;
  margin-left: 10px;
  margin-top: 30px;
}
#details {
  margin-top: 25px;
  margin-left: 220px;
}
#editProfile {
  border: 1px solid #222;
  background-color: white;
  height: 300px;
  width: 800px;
  border-radius: 10px;
  margin-left: 10px;
  margin-top: 30px;
}
#editDetails {
  margin-top: 20px;
  margin-left: 220px;
  line-height: 2px;
}
.editDetails {
  margin: 5px;
}
#changePassword {
  border: 1px solid #222;
  background-color: white;
  height: 300px;
  width: 800px;
  border-radius: 10px;
  margin-left: 10px;
  margin-top: 30px;
}
#editPassword {
  margin-top: 20px;
  margin-left: 220px;
  line-height: 2px;
}
.editPassword {
  margin: 5px;
}
p.name {
  font-size: 40px;
  font-weight: bold;
  margin-top: 0px;
  margin-bottom: 2px;
}
p.details {
  font-size: 20px;
  padding: 1px;
  margin-bottom: 0px;
}
.material-design-icon.editIcon {
  float: right;
  padding: 10px;
}
input {
  font-size: 16px;
  padding: 20px 0px;
  padding-left: 10px;
  height: 56px;
  border: none;
  border-bottom: solid 1px rgba(0, 0, 0, 0.1);
  background: #fff;
  width: 260px;
  box-sizing: border-box;
  transition: all 0.3s linear;
  color: #000;
  font-weight: 400;
  -webkit-appearance: none;
  margin: 0px;
  font-family: sans-serif;
}
input:focus {
  border-bottom: solid 1px black;
  outline: 0;
  box-shadow: 0 2px 6px -8px rgba(rgba(0, 0, 0, 0.1), 0.45);
}
button {
  -webkit-appearance: none;
  width: auto;
  min-width: 100px;
  border-radius: 24px;
  text-align: center;
  padding: 15px 40px;
  margin-top: 23px;
  background-color: black;
  color: #fff;
  font-size: 14px;
  margin-left: 0px;
  margin-right: 30px;
  font-weight: 500;
  box-shadow: 0px 2px 6px -1px rgba(0, 0, 0, 0.13);
  border: none;
  transition: all 0.3s ease;
  outline: 0;
  cursor: pointer;
  font-family: sans-serif;
}
button:hover {
  box-shadow: 0px 2px 6px -1px rgba(0, 0, 0, 0.65);
}
button:active {
  opacity: 0.6;
  transform: translateY(3px);
}
h2 {
  margin-left: 10px;
  margin-bottom: -25px;
  font-size: 30px;
  font-weight: normal;
}
#currentQueue {
  border-top: 3px solid #222;
  background-color: white;
  height: auto;
  width: 100%;
  margin-right: auto;
  margin-top: 30px;
  background-color: #eee;
  padding-bottom: 10px;
}
#currentQueueDetails {
  margin-top: 10px;
  margin-left: 10px;
  padding-bottom: 10px;
}
p.note {
  font-style: italic;
}
#queueHistory {
  border-top: 3px solid #222;
  background-color: white;
  height: auto;
  width: 100%;
  margin-right: auto;
  margin-top: 30px;
  background-color: #eee;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
.notInQ {
  margin-top: 40px;
  margin-left: 10px;
}
button.rate {
  margin-top: 0px;
  margin-bottom: 15px;
}
li.queueHistory {
  text-align: center;
  column-count: 3;
  column-gap: 160px;
}
</style>
