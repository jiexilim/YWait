<template>
  <div>
    <h1>{{ this.name }} Queue System</h1>
    <div class="logout"><LogOutButton /></div>

    <div id="currToCompleted">
      <h2>- Current Queue Number -</h2>
      <table>
        <tr>
          <th>No. Pax</th>
          <th>Current Queue Number</th>
          <th>Customer Name</th>
          <th>Baby Chair</th>
          <th>Wheel Chair</th>
          <th>Additional Message</th>
          <th>Button</th>
        </tr>

        <tr v-for="queue in currentQueueNumber" v-bind:key="queue.id">
          <td>{{ queue["paxGroup"] }}</td>
          <td>{{ queue["queueNumber"] }}</td>
          <td>{{ queue["customerName"] }}</td>
          <td>{{ queue["babyChair"] }}</td>
          <td>{{ queue["wheelChair"] }}</td>
          <td>{{ queue["additionalMessage"] }}</td>
          <td>
            <button
              v-on:click="changeStatus(queue['queueNumber'], 'completed')"
            >
              Have Arrived
            </button>
          </td>
        </tr>
      </table>
    </div>

    <div id="waitingToCurr">
      <h2>- Waiting Queue Numbers -</h2>
      <table>
        <tr>
          <th>No. Pax</th>
          <th>Queue Number</th>
          <th>Customer Name</th>
          <th>Baby Chair</th>
          <th>Wheel Chair</th>
          <th>Additional Message</th>
          <th>Arrival Time</th>
          <th>Button</th>
        </tr>

        <tr v-for="queue in waitingQueue" v-bind:key="queue.id">
          <td>{{ queue["paxGroup"] }}</td>
          <td>{{ queue["queueNumber"] }}</td>
          <td>{{ queue["customerName"] }}</td>
          <td>{{ queue["babyChair"] }}</td>
          <td>{{ queue["wheelChair"] }}</td>
          <td>{{ queue["additionalMessage"] }}</td>
          <td>{{ queue["arrivalTime"] }}</td>
          <td>
            <button v-on:click="changeStatus(queue['queueNumber'], 'current')">
              Push to Current
            </button>
          </td>
        </tr>
      </table>
    </div>
    <div id="charts">
      <zingchart :data="myConfig"></zingchart>
    </div>
  </div>
</template>

<script>
import { database } from "../firebase.js";
import { auth } from "../firebase.js";

export default {
  data() {
    return {
      currentQueueNumber: this.queueData("current"),
      waitingQueue: this.queueData("waiting"),
      name: null,
      rating: null,
      numRatings: null,
      dayTime: [],
    };
  },
  methods: {
    fetchItems: function() {
      database
        .collection("restaurants")
        .doc(`${auth.currentUser.uid}`)
        .get()
        .then((query) => {
          this.rating = query.data().rating;
          this.numRatings = query.data().numRatings;
          this.name = query.data().restaurantName;
        });
    },
    getDataFromQuery: function(query) {
      var data = [];
      query.get().then((snapshot) => {
        snapshot.docs.forEach((doc) => {
          data.push(doc.data());
        });
      });
      return data;
    },
    queueData: function(status) {
      var query = database
        .collection("bookings")
        .where("restaurantId", "==", `${auth.currentUser.uid}`)
        .where("queueStatus", "==", status)
        .orderBy("arrivalTime");
      return this.getDataFromQuery(query);
    },
    changeStatus: function(queueNum, newStatus) {
      database
        .collection("bookings")
        .where("restaurantId", "==", `${auth.currentUser.uid}`)
        .where("queueNumber", "==", queueNum)
        .limit(1)
        .get()
        .then((query) => {
          const queueData = query.docs[0];
          queueData.ref.update({ queueStatus: newStatus }).then(() => {
            location.reload();
          });
        });
    },
    fetchDayTime: function() {
      database
        .collection("bookings")
        .where("restaurantId", "==", auth.currentUser.uid)
        .get()
        .then((snap) => {
          snap.forEach((doc) => {
            this.dayTime.push(doc.data());
          });
        });
    },
    countDayTimeFrequency: function(getDay, getTime) {
      var count = 0
      for (let i = 0; i < this.dayTime.length; i++) {
        if (this.dayTime[i].dayType === getDay && this.dayTime[i].partOfDay === getTime) {
          count += 1
        }
      }
      return count
    }
  },

  computed: {
    computeEmptyRating() {
      var emptyRating = 5 - this.rating;
      return emptyRating;
    },
    ratingChartData() {
      return {
        type: "ring",
        title: {
          text: "- Average Rating -",
          color: "black",
          "font-weight": "bold",
          "font-family": "sans-serif",
          fontSize: "22px",
          "adjust-layout": true,
        },
        scaleR: {
          // Set to half ring
          refAngle: 180,
          aperture: 180,
        },
        tooltip: {
          // Hide tooltip
          visible: false,
        },
        series: [
          {
            // First part of the ring (colored)
            values: [this.rating],
            rules: [
              {
                rule: "this.rating <= 1",
                backgroundColor: "#ff5f5f",
              },
              {
                rule: "1 < this.rating <= 2",
                backgroundColor: "#ffa55a",
              },
              {
                rule: "2 < this.rating <= 3",
                backgroundColor: "#faff5a",
              },
              {
                rule: "3 < this.rating <= 4",
                backgroundColor: "#b9ff55",
              },
              {
                rule: "this.rating > 4",
                backgroundColor: "#73ff5f",
              },
            ],
          },
          {
            // Remainder of ring
            values: [this.computeEmptyRating],
            backgroundColor: "#EDF2F7",
          },
        ],
        plot: {
          slice: "80%", // Ring width,
          detach: false, // Prevent ring piece from detaching on click
          valueBox: [
            {
              // Percentage text
              type: "first",
              text: this.rating.toFixed(2) + " / 5",
              connected: false,
              fontColor: "black",
              "font-family": "sans-serif",
              fontSize: "35px",
              placement: "center",
              visible: true,
              offsetY: "-85px",
            },
            {
              // Label text
              type: "first",
              text: this.numRatings + " Total",
              connected: false,
              fontColor: "black",
              "font-weight": "normal",
              "font-family": "sans-serif",
              fontSize: "25px",
              placement: "center",
              visible: true,
              offsetY: "-35px",
            },
          ],
          animation: {
            effect: "2",
            method: "0",
            speed: "1000",
          },
        },
      };
    },
    DayTimeChartData() {
      return {
        type: "bar",
        "scale-x": {
          labels: ["Sunday", "Monday", "Tuesday","Wednesday","Thursday","Friday","Saturday"],
        },
        "scale-y": {
          step: 1,
        },
        title: {
          text: "- Distribution of Bookings in a Week -",
          color: "black",
          "font-weight": "bold",
          "font-family": "sans-serif",
          fontSize: "22px",
          "adjust-layout": true,
        },
        plotarea: {
          "adjust-layout": true,
        },
        series: [{
          values: [
            this.countDayTimeFrequency(0, 0),
            this.countDayTimeFrequency(1, 0),
            this.countDayTimeFrequency(2, 0),
            this.countDayTimeFrequency(3, 0),
            this.countDayTimeFrequency(4, 0),
            this.countDayTimeFrequency(5, 0),
            this.countDayTimeFrequency(6, 0)], 
            "text": "Morning"
        }, { 
          values: [
            this.countDayTimeFrequency(0, 1),
            this.countDayTimeFrequency(1, 1),
            this.countDayTimeFrequency(2, 1),
            this.countDayTimeFrequency(3, 1),
            this.countDayTimeFrequency(4, 1),
            this.countDayTimeFrequency(5, 1),
            this.countDayTimeFrequency(6, 1)], 
            "text": "Afternoon"
        }, { 
          values: [
            this.countDayTimeFrequency(0, 2),
            this.countDayTimeFrequency(1, 2),
            this.countDayTimeFrequency(2, 2),
            this.countDayTimeFrequency(3, 2),
            this.countDayTimeFrequency(4, 2),
            this.countDayTimeFrequency(5, 2),
            this.countDayTimeFrequency(6, 2)], 
            "text": "Night"
        }],
        plot: {
          tooltip: {
            text: "%t: %vt"
          },
          animation: {
            effect: "2",
            method: "0",
            speed: "1000",
          },
        },
      };
    },
    myConfig() {
      return {
        layout: "horizontal",
        graphset: [
          this.ratingChartData,
          this.DayTimeChartData
        ],
      };
    },
  },
  created() {
    this.fetchItems();
    this.fetchDayTime();
  },
};
</script>

<style scoped>
#wrapper {
  font-family: sans-serif;
}

h1 {
  font-size: 40px;
  font-weight: normal;
  text-align: center;
}

.logout {
  position: absolute;
  top: 2%;
  right: 10%;
}

h2 {
  margin-top: 4%;
  text-align: center;
  font-size: 22px;
}

table,
td,
th {
  border-collapse: collapse;
  text-align: center;
}

table {
  box-shadow: 11px 11px 10px rgba(0, 0, 0, 0.2);
  border-radius: 0 0 10px 10px;
}

#currToCompleted > table {
  width: 70%;
  margin-left: 15%;
  margin-bottom: -1%;
}

#waitingToCurr > table {
  width: 80%;
  margin-left: 10%;
}

th {
  padding: 7px;
  background: url("../../images/waitingQ-dashboard.jpeg");
}

th:first-of-type {
  border-radius: 10px 0 0 0;
}

th:last-of-type {
  border-radius: 0 10px 0 0;
}

button {
  color: black;
  background-color: white;
  border: 2px solid #e7e7e7;
  transition-duration: 0.4s;
  text-align: center;
  align-self: center;
}

button:hover {
  cursor: pointer;
  background-color: #e7e7e7;
}
#charts {
  margin-top: 3%;
}
</style>
