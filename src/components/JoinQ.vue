<template>
  <div>
    <NavBar></NavBar>
    <div id="wrapper">
      <h1>Join a Queue</h1>
      <div id="img">
        <img :src="restaurantImage" />
      </div>
      <div id="form">
        <form @submit.prevent="joinQueue()">
          <p>
            <label for="mall"
              >Choose your preferred dining mall<span style="color: red"
                >*</span
              ></label
            >
          </p>
          <div class="select">
            <select
              v-model="mallSelected"
              v-on:change="selectRestaurants()"
              id="mall"
              required
            >
              <option
                v-for="mall in mallsList"
                v-bind:key="mall.id"
                :value="mall.mallName"
              >
                {{ mall.mallName }}
              </option>
            </select>
          </div>

          <p>
            <label for="restaurant"
              >Choose your preferred dining restaurant<span style="color: red"
                >*</span
              ></label
            >
          </p>
          <div class="select">
            <select v-model="restaurantSelected" required>
              <option
                v-for="restaurant in selectedRestaurantsList"
                v-bind:key="restaurant.id"
                v-bind:value="restaurant"
                required
              >
                {{ restaurant.restaurantName }}
              </option>
            </select>
          </div>

          <p>
            <label for="numAdult"
              >Number of Pax (Adult)<span style="color: red">*</span></label
            >
          </p>
          <p>
            <input
              type="number"
              v-model="numAdult"
              id="numAdult"
              required
              min="0"
            />
          </p>

          <p><label for="numChildren"> Number of Pax (Children) </label></p>
          <p>
            <input
              type="number"
              v-model="numChildren"
              id="numChildren"
              min="0"
            />
          </p>

          <p><label for="babychair"> Number of Baby Chair(s)</label></p>
          <p>
            <input type="number" v-model="babyChair" id="babychair" min="0" />
          </p>

          <p><label for="wheelchair"> Number of Wheelchair(s)</label></p>
          <p>
            <input type="number" v-model="wheelChair" id="wheelchair" min="0" />
          </p>

          <p><label for="additionalMessage">Message</label></p>
          <p>
            <textarea
              v-model="additionalMessage"
              id="additionalMessage"
              rows="4"
              cols="50"
              placeholder="Any additonal message"
            ></textarea>
          </p>

          <p>
            <b>* Full party</b> must be present for
            <span style="color: red"
              >at least 5 minutes before reservation time</span
            >
            to be seated.<br />We regret to inform that queue seatings will be
            given up if you are late due to the limited seating capacity.<br />
            Kindly double-click the 'Join Queue' button to confirm your reservation.
          </p>

          <div id="button">
            <button type="submit">Join Queue</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { database } from "../firebase.js";
import { auth } from "../firebase.js";

export default {
  data() {
    return {
      // all details from database
      mallsList: ["All"],
      restaurantsList: [],
      selectedRestaurantsList: [],

      //FORM
      customerID: `${auth.currentUser.uid}`,
      customerName: "",
      mallSelected: "all",
      restaurantSelected: "",
      numAdult: null,
      numChildren: null,
      babyChair: null,
      wheelChair: null,
      additionalMessage: "",

      //image
      restaurantImage: require("../../images/joinQ-restaurant.jpg"),

      queueNum: 0,
      queueNumId: "",
      dayType: 0,
      partOfDay: 0,
      freq: "",
      arrTime: "",
    };
  },
  methods: {
    joinQueue: function () {
      var paxGroup = this.getPaxGroup(this.numAdult, this.numChildren);
      var restName = this.restaurantSelected.restaurantName;
      var mallName = this.mallSelected;
      this.getArrivalTime(paxGroup, restName, mallName);
    },
    getCurrentTime: function () {
      const curr = new Date();
      return curr.toString();
    },
    getArrivalTime(paxGroup, restName, mallName) {
      let counter = 0;
      database
        .collection("bookings")
        .get()
        .then((snapshot) => {
          snapshot.docs.forEach((doc) => {
            var booking = doc.data();
            if (
              booking.paxGroup === paxGroup &&
              booking.restaurantName === restName &&
              booking.mallName === mallName &&
              booking.queueStatus === "waiting"
            ) {
              counter += 1;
            }
          });
        })
        .then(() => {
          var bookTime = this.getCurrentTime();
          var parsed = Date.parse(bookTime) + counter * 2700000;
          this.dayType = new Date(parsed).getDay();
          this.arrTime = new Date(parsed).toString();
          this.partOfDay = this.getPartOfDay(new Date(parsed).getHours());
          database.collection("bookings").add({
            customerID: this.customerID,
            customerName: this.customerName,
            mallName: mallName,
            restaurantName: restName,
            restaurantMall: restName + " @ " + mallName,
            numAdult: this.numAdult,
            numChildren: this.numChildren,
            paxGroup: paxGroup,
            queueNumber: this.getQueueNumber(this.numAdult, this.numChildren),
            babyChair: this.babyChair,
            wheelChair: this.wheelChair,
            additionalMessage: this.additionalMessage,
            queueStatus: "waiting",
            bookedTiming: bookTime,
            restaurantId: this.restaurantSelected.id,
            arrivalTime: this.arrTime,
            // 0 for SUN, 1 for MON, 2 for TUES ...
            dayType: this.dayType,
            partOfDay: this.partOfDay,
          });
          alert(
            "Successfully joined queue. Please arrive at " +
              restName +
              " @ " +
              mallName +
              " by " +
              this.arrTime +
              "."
          );
          this.$router.push("/myprofile");
        });
    },
    getPartOfDay: function(hrs) {
        // 0 for morning, 1 for afternoon, 2 for evening
        if (hrs < 12) {
            return 0
        } else if (hrs < 18) {
            return 1
        } else {
            return 2
        }
    },
    calculatePax: function(numAdult, numChildren) {
      let children = 0;
      if (numChildren !== null) {
        children = numChildren;
      }
      return parseInt(numAdult, 10) + children;
    },
    getPaxGroup: function (numAdult, numChildren) {
      const totalPax = this.calculatePax(numAdult, numChildren);
      var paxGroup;
      if (totalPax < 3) paxGroup = "1 to 2 people";
      else if (totalPax < 5) paxGroup = "3 to 4 people";
      else paxGroup = "5 or more people";
      return paxGroup;
    },
    getQueueNumber: function (numAdult, numChildren) {
      const totalPax = this.calculatePax(numAdult, numChildren);
      var qCategory;
      if (totalPax < 3) qCategory = "A";
      else if (totalPax < 5) qCategory = "B";
      else qCategory = "C";

      database
        .collection("queueNumbers")
        .get()
        .then((snapshot) => {
          snapshot.docs.forEach((doc) => {
            if (qCategory === "A") {
              this.queueNum = doc.data().queueNumA;
              this.queueNumId = doc.id;
            } else if (qCategory === "B") {
              this.queueNum = doc.data().queueNumB;
              this.queueNumId = doc.id;
            } else {
              this.queueNum = doc.data().queueNumC;
              this.queueNumId = doc.id;
            }
          });
        });
      this.queueNum += 1;
      var num = String(this.queueNum);
      while (num.length < 4) num = "0" + num;
      var res = qCategory + num;

      if (qCategory === "A") {
        database.collection("queueNumbers").doc(this.queueNumId).update({
          queueNumA: this.queueNum,
        });
      } else if (qCategory === "B") {
        database.collection("queueNumbers").doc(this.queueNumId).update({
          queueNumB: this.queueNum,
        });
      } else {
        database.collection("queueNumbers").doc(this.queueNumId).update({
          queueNumC: this.queueNum,
        });
      }
      return res;
    },
    // getDetails is for when created stage
    getDetails: function () {
      // get mallsList
      database
        .collection("malls")
        .get()
        .then((snapshot) => {
          snapshot.docs.forEach((doc) => {
            var curr = doc.data();
            this.mallsList.push(curr);
          });
        });
      // get restaurantsList
      database
        .collection("restaurants")
        .get()
        .then((snapshot) => {
          snapshot.docs.forEach((doc) => {
            var curr = doc.data();
            curr.id = doc.id;
            this.restaurantsList.push(curr);
          });
        });
      // get customerName
      database
        .collection("users")
        .doc(`${auth.currentUser.uid}`)
        .get()
        .then((querySnapShot) => {
          this.customerName = querySnapShot.data().name;
        });
    },
    exploreMall: function () {
      if (typeof this.$route.params.id !== "undefined") {
        this.mallSelected = String(this.$route.params.id);
        //alert(this.mallSelected);
      }
      this.selectRestaurants();
    },
    selectRestaurants: function () {
      for (const rest of this.restaurantsList) {
        if (rest.mallName == this.mallSelected) {
          this.selectedRestaurantsList.push(rest);
        }
      }
    },
  },
  created() {
    this.getDetails();
    setTimeout(() => {
      this.exploreMall();
    }, 400);
  },
};
</script>

<style scoped>
#wrapper {
  margin-left: 300px;
  font-family: sans-serif;
}
h1 {
  text-align: center;
  font-size: 40px;
  font-weight: normal;
}
#form {
  display: block;
  text-align: center;
}
form {
  display: inline-block;
  margin-left: auto;
  margin-right: auto;
  text-align: left;
  font-size: 18px;
  margin-top: -650px;
  font-family: sans-serif;
}
select {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;

  display: block;
  width: 100%;
  max-width: 320px;
  height: 50px;
  margin: 5px 0px;
  padding: 0px 24px;
  font-size: 16px;
  line-height: 1.75;
  color: #333;
  background-color: #ffffff;
  background-image: none;
  border: 1px solid rgba(0, 0, 0, 0.1);
  -ms-word-break: normal;
  word-break: normal;
  border-radius: 10px;
}

.select:after {
  content: ">";
  text-indent: 10px;
  font: 17px "Consolas", monospace;
  color: black;
  -webkit-transform: rotate(90deg);
  -moz-transform: rotate(90deg);
  -ms-transform: rotate(90deg);
  transform: rotate(90deg);
  padding: 0 0 10px;
  border-bottom: 1px solid #999;
  width: 30px;
  position: absolute;
  pointer-events: none;
  margin-left: 275px;
  margin-top: -45px;
}

select:focus {
  border: solid 1px black;
  outline: 0;
  box-shadow: 0 2px 6px -8px rgba(rgba(0, 0, 0, 0.1), 0.45);
}

#numAdult {
  font-size: 16px;
  padding-bottom: 5px;
  padding-left: 10px;
  height: 30px;
  border: none;
  border-bottom: solid 1px rgba(0, 0, 0, 0.1);
  background: #fff;
  width: 280px;
  box-sizing: border-box;
  transition: all 0.3s linear;
  color: #000;
  font-weight: 400;
  -webkit-appearance: none;
}

#numChildren {
  font-size: 16px;
  padding-bottom: 5px;
  padding-left: 10px;
  height: 30px;
  border: none;
  border-bottom: solid 1px rgba(0, 0, 0, 0.1);
  background: #fff;
  width: 280px;
  box-sizing: border-box;
  transition: all 0.3s linear;
  color: #000;
  font-weight: 400;
  -webkit-appearance: none;
}

#babychair {
  font-size: 16px;
  padding-bottom: 5px;
  padding-left: 10px;
  height: 30px;
  border: none;
  border-bottom: solid 1px rgba(0, 0, 0, 0.1);
  background: #fff;
  width: 280px;
  box-sizing: border-box;
  transition: all 0.3s linear;
  color: #000;
  font-weight: 400;
  -webkit-appearance: none;
}

#wheelchair {
  font-size: 16px;
  padding-bottom: 5px;
  padding-left: 10px;
  height: 30px;
  border: none;
  border-bottom: solid 1px rgba(0, 0, 0, 0.1);
  background: #fff;
  width: 280px;
  box-sizing: border-box;
  transition: all 0.3s linear;
  color: #000;
  font-weight: 400;
  -webkit-appearance: none;
}

#numAdult:focus {
  border-bottom: solid 1px black;
  outline: 0;
  box-shadow: 0 2px 6px -8px rgba(rgba(0, 0, 0, 0.1), 0.45);
}

#numChildren:focus {
  border-bottom: solid 1px black;
  outline: 0;
  box-shadow: 0 2px 6px -8px rgba(rgba(0, 0, 0, 0.1), 0.45);
}

#babychair:focus {
  border-bottom: solid 1px black;
  outline: 0;
  box-shadow: 0 2px 6px -8px rgba(rgba(0, 0, 0, 0.1), 0.45);
}

#wheelchair:focus {
  border-bottom: solid 1px black;
  outline: 0;
  box-shadow: 0 2px 6px -8px rgba(rgba(0, 0, 0, 0.1), 0.45);
}

textarea {
  font-size: 16px;
  padding: 10px 0px;
  padding-left: 10px;
  border: solid 1px rgba(0, 0, 0, 0.1);
  background: #fff;
  box-sizing: border-box;
  transition: all 0.3s linear;
  color: #000;
  -webkit-appearance: none;
  border-radius: 10px;
}

textarea:focus {
  border: solid 1px black;
  outline: 0;
  box-shadow: 0 2px 6px -8px rgba(rgba(0, 0, 0, 0.1), 0.45);
}

textarea::placeholder {
  font-family: sans-serif;
}

#button {
  text-align: center;
}

button {
  -webkit-appearance: none;
  width: auto;
  min-width: 100px;
  border-radius: 24px;
  text-align: center;
  padding: 15px 40px;
  margin-top: 5px;
  background-color: black;
  color: #fff;
  font-size: 16px;
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

#img {
  display: block;
  text-align: center;
}
img {
  display: inline-block;
  width: 400px;
  margin-bottom: 90px;
  margin-top: 10px;
  margin-left: 30%;
  border-radius: 10px;
  box-shadow: 10px 10px 20px rgba(0, 0, 0, 0.3);
}
</style>
