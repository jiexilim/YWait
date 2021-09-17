<template>
  <div class="sidenav">
    <nav>
      <img :src="logo" />
      <ul>
        <li>
          <homeIcon class="icon" /><router-link to="/home" exact
            >Home</router-link
          >
        </li>
        <br />
        <li>
          <clockIcon class="icon" />
          <router-link to="/waitingqueue" exact>Waiting Queues</router-link>
        </li>
        <br />
        <li>
          <mapIcon class="icon" /><router-link to="/malls" exact
            >Malls</router-link
          >
        </li>
        <br />
        <li>
          <calendarIcon class="icon" /><router-link to="/joinq" exact
            >Join a Queue</router-link
          >
        </li>
        <br />
        <li>
          <profileIcon class="icon" /><router-link to="/myprofile" exact
            >My Profile</router-link
          >
        </li>
        <br /><br /><br />
      </ul>
      <chatIcon class="chatIcon" :size="50" />
      <div class="help">
        <h2>Require any assistance?</h2>
        <p class="email">Email us at:</p>
        <p class="emailLink">
          <a href="mailto:help@YWait.com">help@YWait.com</a>
        </p>
      </div>
      <div class="logout"><LogOutButton /></div>
    </nav>
  </div>
</template>

<script>
import homeIcon from "vue-material-design-icons/HomeOutline";
import clockIcon from "vue-material-design-icons/ClockOutline";
import mapIcon from "vue-material-design-icons/MapMarkerOutline";
import calendarIcon from "vue-material-design-icons/CalendarMonthOutline";
import profileIcon from "vue-material-design-icons/AccountCircleOutline";
import chatIcon from "vue-material-design-icons/ChatProcessingOutline";
export default {
  data() {
    return {
      logo: require("../../images/logo.png"),
    };
  },
  components: {
    homeIcon,
    clockIcon,
    mapIcon,
    calendarIcon,
    profileIcon,
    chatIcon,
  },
};
</script>

<style scoped>
.sidenav {
  font-family: sans-serif;
}
img {
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 90%;
  margin-bottom: -50px;
  margin-top: -60px;
}
nav {
  height: 100%;
  width: 300px;
  position: fixed;
  top: 0;
  left: 0;
  padding-top: 40px;
  background-color: #eee;
  overflow-x: hidden;
  font-size: 20px;
  text-align: center;
}
a {
  text-decoration: none;
  color: black;
}
a:hover {
  color: grey;
}
a.router-link-exact-active {
  color: grey;
}
.material-design-icon.icon {
  margin-right: 10px;
  display: inline-flex;
  align-self: center;
  position: relative;
  bottom: -0.225em;
}
li {
  text-align: left;
  margin-right: 40px;
  color: black;
  padding: 10px;
}
.material-design-icon.chatIcon {
  display: inline-flex;
  align-self: center;
  position: relative;
  margin-top: 40px;
}
div.help {
  padding: 10px 15px 10px;
  width: 220px;
  border-radius: 10px;
  background-color: white;
  margin-left: auto;
  margin-right: auto;
  margin-top: -10px;
}
h2 {
  font-size: 17px;
}
p.email {
  font-size: 15px;
}
p.emailLink {
  font-size: 15px;
  margin-top: -10px;
  text-decoration: underline;
  color: blue;
}
div.logout {
  margin-top: 5%;
}
ul {
  list-style-type: none;
}
</style>
