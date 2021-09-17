<template>
  <div>
    <NavBar></NavBar>
    <div id="wrapper">
      <h1>Waiting Queue</h1>

      <div id="filterBar">
        <select v-model="filterMall" id="filterMall">
          <option value="All" hidden>Select Mall</option>
          <option
            v-for="mall in mallsList"
            v-bind:key="mall.id"
            id="selectMallOption"
          >
            {{ mall }}
          </option>
        </select>

        <input
          type="text"
          v-model="searchRestaurant"
          placeholder="Search Restaurant Name"
          id="filterRestaurant"
        />

      </div>

      <div id="summary">
        <div id="numRestaurant">
          <p id="numRestaurantVal">{{ summaryData.length }}</p>
          <p>Restaurants in {{ location }}</p>
        </div>
        <div id="note"></div>
      </div>

      <div id="dashboard">
        <ul>
          <li v-for="(value, key) in getGroupedData()" v-bind:key="key">
            <h3 style="padding-left: 20px">{{ key }}</h3>
            <table>
              <tr>
                <th>No. Pax</th>
                <th>Queue Number</th>
                <th>No. Groups Waiting</th>
                <th>Estimated Waiting Time</th>
              </tr>

              <tr>
                <td>1 to 2 people</td>
                <td>{{ getQueueNumber(key, "1 to 2 people") }}</td>
                <td>{{ getNumWaiting(getQueueNumber(key, "1 to 2 people"), value["1 to 2 people"] || 0) }}</td>
                <td>{{ (getNumWaiting(getQueueNumber(key, "1 to 2 people"), value["1 to 2 people"] || 0)) * 5 }} minutes</td>
              </tr>

              <tr>
                <td>3 to 4 people</td>
                <td>{{ getQueueNumber(key, "3 to 4 people") }}</td>
                <td>{{ getNumWaiting(getQueueNumber(key, "3 to 4 people"), value["3 to 4 people"] || 0) }}</td>
                <td>{{ (getNumWaiting(getQueueNumber(key, "3 to 4 people"), value["3 to 4 people"] || 0)) * 6 }} minutes</td>
              </tr>

              <tr>
                <td>5 or more people</td>
                <td>{{ getQueueNumber(key, "5 or more people") }}</td>
                <td>{{ getNumWaiting(getQueueNumber(key, "5 or more people"), value["5 or more people"] || 0) }}</td>
                <td>{{ (getNumWaiting(getQueueNumber(key, "5 or more people"), value["5 or more people"] || 0)) * 8 }} minutes</td>
              </tr>
            </table>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { database } from "../firebase.js";

export default {
  data() {
    return {
      // all details from database
      mallsList: ["All"],
      currentQueue: [],
      // filter starts here
      filterMall: null,
      searchRestaurant: null,
      sortBy: "asc",
      // filtered data using getFilteredData method
      displayedData: [],
      // summary
      summaryData: [],
      location: "",
    };
  },
  watch: {
    filterMall: function () {
      var query = this.filterQuery();
      this.displayedData = this.getDataFromQuery(query);
      var query2;
      if (this.filterMall == "All") {
        this.location = "Total";
        query2 = database.collection("restaurants");
      } else {
        this.location = this.filterMall;
        query2 = database
          .collection("restaurants")
          .where("mallName", "==", this.filterMall);
      }
      this.summaryData = this.getDataFromQuery(query2);
    },
    searchRestaurant: function () {
      var query = this.filterQuery();
      this.displayedData = this.getDataFromQuery(query);
    },
  },
  methods: {
    // for orderBy restaurantName (asc, desc)
    changeOrder: function () {
      if (this.sortBy == "asc") {
        this.sortBy = "desc";
      } else {
        this.sortBy = "asc";
      }
      var query = this.filterQuery();
      this.displayedData = this.getDataFromQuery(query);
    },
    // for displayedData
    filterQuery: function () {
      var query = database
        .collection("bookings")
        .where("queueStatus", ">", "completed");
      query = query.orderBy("queueStatus");
      if (this.filterMall != "All") {
        query = query.where("mallName", "==", this.filterMall);
      }
      if (this.searchRestaurant) {
        query = query.where("restaurantName", "==", this.searchRestaurant);
      }
      return query;
    },
    getDataFromQuery: function (query) {
      var data = [];
      query.get().then((snapshot) => {
        snapshot.docs.forEach((doc) => {
          data.push(doc.data());
        });
      });
      return data;
    },
    // display on WaitingQueue page: for displayedData
    getGroupedData: function () {
      const groupAll = (list) =>
        list.reduce((data, doc) => {
          const numWait = data[doc.restaurantMall] || 0;
          if (numWait == 0) {
            // when restaurantMall key does not exist
            return Object.assign({}, data, {
              [doc.restaurantMall]: { [doc.paxGroup]: numWait + parseInt(1) },
            });
          } else {
            // when restaurantMall key exists
            const numWait2 = data[doc.restaurantMall][doc.paxGroup] || 0;
            var obj = Object.assign({}, data[doc.restaurantMall], {
              [doc.paxGroup]: numWait2 + parseInt(1),
            });
            return Object.assign({}, data, { [doc.restaurantMall]: obj });
          }
        }, {});
      return groupAll(this.displayedData);
    },
    getQueueNumber: function (restaurantMall, paxGroup) {
      for (var i = 0; i < this.currentQueue.length; i++) {
        if (
          this.currentQueue[i].restaurantMall === restaurantMall &&
          this.currentQueue[i].paxGroup === paxGroup
        ) {
          return this.currentQueue[i].queueNumber;
        }
      }
      return "-";
    },
    getNumWaiting: function(queueNumVal, currAndWaitNum) {
      if (queueNumVal === "-") {
        return currAndWaitNum
      } else {
        return currAndWaitNum - 1
      }
    },

    fetchData: function () {
      // get mallsList
      database
        .collection("malls")
        .get()
        .then((snapshot) => {
          snapshot.docs.forEach((doc) => {
            var curr = doc.data();
            this.mallsList.push(curr.mallName);
          });
        });
      // get current Queue List
      this.filterMall = this.mallsList[0];
      database
        .collection("bookings")
        .where("queueStatus", "==", "current")
        .get()
        .then((snapshot) => {
          snapshot.docs.forEach((doc) => {
            this.currentQueue.push(doc.data());
          });
        });
    },
  },
  created() {
    this.fetchData();
  },
};
</script>

<style scoped>
#wrapper {
  margin-left: 300px;
  font-family: Helvetica, sans-serif;
}

h1 {
  font-size: 40px;
  font-weight: normal;
  text-align: center;
}

#filterBar {
  display: block;
  margin-top: 2%;
  margin-left: 10%;
  height: 40px;
}

#filterBar > #filterMall {
  width: 20%;
  padding: 7px 14px;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 5%;
}

#filterBar > #filterRestaurant {
  width: 35%;
  padding: 7px 14px;
}

#summary {
  margin-top: 3%;
  display: flex;
}

#summary > #numRestaurant {
  display: inline-block;
  padding: 7px;
  width: 24%;
  height: 150px;
  box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  margin-left: 9%;
  text-align: center;
  align-self: center;
  background: url(../../images/waitingQ-summary.jpeg);
  background-repeat: no-repeat;
  background-size: 100% 100%;
}

#summary > #numRestaurant > #numRestaurantVal {
  font-size: 30px;
  font-weight: bold;
}

#summary > #note {
  display: inline-block;
  padding: 7px 25px;
  width: 50%;
  height: 150px;
  box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  margin-left: 3%;
  background: url("../../images/waitingQ-summary2.jpeg");
  background-repeat: no-repeat;
  background-size: 100% 300%;
  background-position: bottom;
}

#dashboard {
  display: block;
}

#dashboard > ul {
  list-style-type: none;
  width: 85%;
  margin-left: 4%;
}

#dashboard > ul > li {
  padding-top: 5px;
  margin-top: 5%;
  box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  background: url("../../images/waitingQ-dashboard.jpeg");
  background-repeat: no-repeat;
  background-size: 150% 150%;
}

#dashboard > ul > li > table {
  width: 100.1%;
  padding: 7px;
  background-color: white;
  border-bottom-left-radius: 10px;
  border-bottom-right-radius: 10px;
}

#dashboard > ul > li > table > tr > th,
td {
  text-align: center;
  padding: 14px;
}
</style>
