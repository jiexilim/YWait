<template>
  <div>
    <NavBar></NavBar>
    <div id="wrapper">
      <h1>Hi {{ this.name }}!</h1>
      <div id="joinQ">
        <router-link to="/joinq" exact>Join a Queue Here!</router-link>
      </div>
      <p></p>
      <div id="mallCalendar">
        <div id="viewMalls">
          <router-link to="/malls" exact>View Malls</router-link>
        </div>
        <Calendar id="calendar"></Calendar>
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
      name: "",
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
        });
    },
  },
  created() {
    this.fetchItems();
  },
};
</script>

<style scoped>
#wrapper {
  margin-left: 330px;
  font-family: sans-serif;
}
h1 {
  text-align: left;
  margin-top: 0px;
  margin-bottom: 10px;
  font-size: 70px;
}

#joinQ {
  padding: 7px 25px;
  width: 95%;
  height: 350px;
  box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  background: url("../../images/home-page-dashboard.jpeg");
  background-repeat: no-repeat;
  background-size: 100% 300%;
  background-position: bottom;
  font-weight: bold;
  color: white;
  text-align: left;
  font-size: 40px;
  text-shadow: 2px 2px black;
  padding-top: 10px;
}

#mallCalendar {
  display: flex;
  height: 450px;
}

#viewMalls {
  display: inline-block;
  padding: 7px 25px;
  vertical-align: top;
  width: 45%;
  box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  background: url("../../images/home-page-view.jpeg");
  background-repeat: no-repeat;
  background-position: center;
  background-size: 100% 100%;
  color: white;
  font-weight: bold;
  text-align: left;
  font-size: 40px;
  text-shadow: 2px 2px black;
  padding-top: 10px;
}

#calendar {
  display: inline-block;
  padding: 50px 25px;
  width: 44.55%;
  margin-left: 2%;

  box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
}

a {
  text-decoration: none;
  color: white;
}

a:visited {
  color: white;
}

a:link {
  color: white;
}

a:hover {
  color: rgb(201, 199, 199);
}
</style>
