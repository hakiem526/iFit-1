<template>
  <div id="background">
    <b-card id="form" bg-variant="light">
      <h1><b> Register</b></h1>
      <br />
      <form
        @submit.prevent="register"
        oninput='pw2.setCustomValidity(pw2.value != pw1.value ? "Passwords do not match." : "")'
      >
        <b-form-group
          class="attribute"
          id="input-group-1"
          label="Your Name:"
          label-for="input-1"
          label-cols-sm="3"
        >
          <b-form-input
            id="input-1"
            v-model="name"
            placeholder="Enter name"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group
          class="attribute"
          id="input-group-2"
          label="Email address:"
          label-for="input-2"
          label-cols-sm="3"
          description="Please enter a valid email address."
        >
          <b-form-input
            id="input-2"
            v-model="email"
            type="email"
            placeholder="Enter email"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group
          class="attribute"
          id="input-group-3"
          label="Telegram:"
          label-for="input-3"
          label-cols-sm="3"
        >
          <b-form-input
            id="input-3"
            v-model="tele"
            placeholder="@username"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group
          class="attribute"
          id="input-group-4"
          label="Height:"
          label-for="input-4"
          label-cols-sm="3"
        >
          <b-form-input
            id="input-4"
            v-model="height"
            placeholder="Enter height (in m)"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group
          class="attribute"
          id="input-group-5"
          label="Weight:"
          label-for="input-5"
          label-cols-sm="3"
        >
          <b-form-input
            id="input-5"
            v-model="weight"
            placeholder="Enter weight (in kg)"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group
          class="attribute"
          id="input-group-6"
          label="Your Age:"
          label-for="input-6"
          label-cols-sm="3"
        >
          <b-form-input
            id="input-6"
            v-model="age"
            placeholder="Enter age"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group
          class="attribute"
          id="input-group-7"
          label="Your desired calories burnt target:"
          label-for="input-7"
          label-cols-sm="3"
          description="Desired calories burnt per month"
        >
          <b-form-input
            id="input-7"
            v-model="goal"
            placeholder="Enter a value"
            required
          ></b-form-input>
        </b-form-group>

        <b-form-group
          class="attribute"
          id="input-group-8"
          label="Password:"
          label-for="pswd"
          label-cols-sm="3"
        >
          <b-form-input
            id="pswd"
            type="password"
            name="pw1"
            v-model="password"
            :state="valid"
            placeholder="Enter a password"
            required
          ></b-form-input>
          <b-form-invalid-feedback :state="valid">
            <span :class="has_minimum_length ? 'has_required' : ''"
              >Your password should contain at least 8 characters.</span
            >
            <br /><span :class="has_lowercase ? 'has_required' : ''"
              >Must have at least 1 LOWER-case letter</span
            >
            <br /><span :class="has_uppercase ? 'has_required' : ''"
              >Must have at least 1 UPPER-case letter</span
            >
            <br /><span :class="has_number ? 'has_required' : ''"
              >Must contain at least 1 number.</span
            >
            <br /><span :class="has_special ? 'has_required' : ''"
              >Must contain a special character.</span
            >
          </b-form-invalid-feedback>
          <b-form-valid-feedback :state="valid">
            Looks Good!
          </b-form-valid-feedback>
        </b-form-group>

        <b-form-group
          class="attribute"
          id="input-group-9"
          label="Confirm Password:"
          label-for="pswd2"
          label-cols-sm="3"
        >
          <b-form-input
            id="pswd2"
            type="password"
            name="pw2"
            placeholder="Enter your password again"
            required
          ></b-form-input>
        </b-form-group>

        <br /><b-button class="submitBut" type="submit" variant="warning" size="lg"
          ><b>SIGN UP</b></b-button
        >
        <br /><br />
        <p class="loginQues">
          Already have an account? Log in
          <router-link to="/login" id="login-link"><b>here</b></router-link
          >.
        </p>
      </form>
    </b-card>
    <Footer></Footer>
  </div>
</template>

<script>
import firebase from "firebase";
import database from "../firebase";
import Footer from './Footer.vue'

export default {
  components: {
    Footer
  },
  data() {
    return {
      email: "",
      name: "",
      tele: "",
      height: "",
      weight: "",
      age: "",
      password: "",
      goal: "",

      has_minimum_length: false,
      has_number: false,
      has_lowercase: false,
      has_uppercase: false,
      has_special: false,
    };
  },
  watch: {
    password() {
      const format = /[!@#$%^&*()_+\-=\]{};':"\\|,.<>?]/;
      this.has_minimum_length = this.password.length >= 8;
      this.has_number = /\d/.test(this.password);
      this.has_lowercase = /[a-z]/.test(this.password);
      this.has_uppercase = /[A-Z]/.test(this.password);
      this.has_special = format.test(this.password);
    },
  },
  computed: {
    valid() {
      return (
        this.has_minimum_length &&
        this.has_special &&
        this.has_uppercase &&
        this.has_lowercase &&
        this.has_number
      );
    },
  },
  methods: {
    register() {
      if (
        this.has_minimum_length &&
        this.has_special &&
        this.has_uppercase &&
        this.has_lowercase &&
        this.has_number
      ) {
        firebase
          .auth()
          .createUserWithEmailAndPassword(this.email, this.password)
          .then(() => {
            this.sendVerification();
            var joinDate = new Date(); //access date when the user first joins iFit
            database
              .collection("user")
              .doc(firebase.auth().currentUser.uid)
              .set({
                name: this.name,
                email: this.email,
                tele: this.tele,
                height: Number(this.height),
                weight: Number(this.weight),
                age: Number(this.age),
                password: this.password,
                goal: Number(this.goal),
                startDate: joinDate.getDate(),
                startMonth: joinDate.getMonth() + 1,
                startYear: joinDate.getFullYear(),
                burnt: 0,
                showTele: true,
                showPromo: false,
                showMilestones: false,
                showClassAvail: false,
                valid: false,
                profilePic: 'https://image.freepik.com/free-vector/cute-smiling-happy-strong-avocado-with-jumping-rope-flat-cartoon-character-illustration-icon-isolated-white-avocado-gym-lifestyle-sport-jump-rope-health-fitness-nutrition_92289-524.jpg'
              });
            //alert('Successfully registered! Please login.');
            this.$router.push("/verify");
          })
          .catch((error) => {
            alert(error.message);
          });
      } else {
        alert("Password does not meet minimum criteria, please try again.");
      }
    },
    sendVerification() {
      var actionCodeSettings = {
        // to change when deployed using firebase
        url: "http://localhost:8080/login",
        handleCodeInApp: true,
      };
      firebase.auth().onAuthStateChanged((firebaseUser) => {
        if (firebaseUser) {
          firebaseUser
            .sendEmailVerification(actionCodeSettings)
            .then(function () {
              firebase
                .auth()
                .signOut()
                .then(() => {
                  // Sign-out successful.
                })
                .catch((error) => {
                  console.log(error);
                });
            });
        }
      });
    },
  },
};
</script>

<style scoped>
#background {
  background: url(/static/signup_login_background.jpg);
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 130%;
  background-size: cover;
}
#form {
  width: 60%;
  height: 87%;
  margin-top: 5rem;
  margin-left: auto;
  margin-right: auto;
  border: 3px solid rgb(95, 93, 93);
  border-radius: 20px;
  background-color: rgb(207, 203, 203);
}
#title {
  font-size: 20rem;
}
label {
  text-align: left;
}
#login-link {
  color: rgb(255, 208, 0);
  font-family: 'Rubik', sans-serif;
}
.has_required {
  text-decoration: line-through;
  color: #689868;
}
h1 {
  font-family: 'Fjalla One', sans-serif;
}
.attribute {
  font-family: 'Rubik', sans-serif;
}
.loginQues {
  font-family: 'Rubik', sans-serif;
}
.submitBut {
  font-family: 'Fjalla One', sans-serif;
}
Footer {
  position: absolute;
  top: 100%;
  width: 100%;
}
</style>