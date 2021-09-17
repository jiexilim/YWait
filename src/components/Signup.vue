<template>
  <div id="wrapper">
    <div class="card card-container">
      <img :src="logo" />
      <p>A Virtual Online Queueing System</p>
      <form id="signup" @submit.prevent="signup()">
        <p>
          <input
            type="text"
            name="name"
            id="name"
            v-model="name"
            placeholder="Name"
            required
          />
        </p>
        <p>
          <input
            type="email"
            name="email"
            id="email"
            v-model="email"
            placeholder="Email Address"
            required
          />
        </p>
        <p>
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
        <p>
          <input
            type="date"
            name="dob"
            id="dob"
            v-model="dob"
            placeholder="Date of Birth"
            required
          />
        </p>
        <p>
          <input
            type="password"
            name="password"
            id="password"
            v-model="password"
            placeholder="Password"
            required
          />
        </p>
        <p>
          <input
            type="password"
            name="confirmPassword"
            id="confirmPassword"
            v-model="confirmPassword"
            placeholder="Confirm Password"
            required
          />
        </p>
        <button type="submit">Sign Up</button><br /><br />
        Have an account? Login <router-link to="/" exact>here</router-link>!
      </form>
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
      email: "",
      contact: null,
      dob: null,
      password: "",
      confirmPassword: "",
      logo: require("../../images/logo.png"),
    };
  },
  methods: {
    signup() {
      if (this.password.length < 6) {
        alert(
          "Password must contain at least 6 characters. Please check your password and try again."
        );
      } else if (this.password != this.confirmPassword) {
        alert(
          "Passwords do not match. Please check your password and try again."
        );
      } else {
        auth
          .createUserWithEmailAndPassword(this.email, this.password)
          .then(() => {
            if (this.password != this.confirmPassword) {
              throw new Error();
            } else {
              alert("Successfully signed up!");
              this.$router.replace({ name: "Login" });
              this.registerUser();
            }
          })
          .catch((error) => {
            if (error.code == "auth/invalid-email") {
              alert(
                "The email you entered is invalid. Please check your email and try again."
              );
            } else if (error.code == "auth/email-already-in-use") {
              alert(
                "Email already exists. Please enter a different email and try again."
              );
            }
          });
      }
    },
    registerUser() {
      database
        .collection("users")
        .doc(`${auth.currentUser.uid}`)
        .set({
          customerID: auth.currentUser.uid,
          name: this.name,
          email: auth.currentUser.email,
          contact: this.contact,
          dob: this.dob,
        });
    },
  },
};
</script>

<style scoped>
#wrapper {
  background-image: url("http://tourgeorgia.ge/wp-content/uploads/2017/02/HD-Jasmin-Restaurant.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  background-color: rgba(255, 255, 255, 0.5);
  background-blend-mode: lighten;
  min-height: 100%;
  min-width: 1024px;
  width: 100%;
  height: auto;
  position: fixed;
  top: 0;
  left: 0;
  background-blend-mode: overlay;
  margin-left: 0px;
  font-family: sans-serif;
}
.card-container.card {
  max-width: 350px !important;
  padding: 40px 40px;
  margin-top: 80px;
}
.card {
  background-color: #f7f7f7;
  padding: 20px 25px 30px;
  margin: 0 auto 25px;
  margin-top: 50px;
  -moz-border-radius: 2px;
  -webkit-border-radius: 2px;
  border-radius: 10px;
  -moz-box-shadow: 0px 2px 2px rgba(0, 0, 0, 0.3);
  -webkit-box-shadow: 0px 2px 2px rgba(0, 0, 0, 0.3);
  box-shadow: 0px 2px 2px rgba(0, 0, 0, 0.3);
  text-align: center;
}
img {
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 90%;
  margin-bottom: -50px;
  margin-top: -60px;
}
input {
  font-size: 16px;
  padding: 20px 0px;
  padding-left: 10px;
  height: 56px;
  border: none;
  border-bottom: solid 1px rgba(0, 0, 0, 0.1);
  background: #fff;
  width: 280px;
  box-sizing: border-box;
  transition: all 0.3s linear;
  color: #000;
  font-weight: 400;
  -webkit-appearance: none;
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
  margin-top: 5px;
  background-color: black;
  color: #fff;
  font-size: 14px;
  margin-left: auto;
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
a {
  color: black;
}
a:hover {
  color: grey;
}
</style>
