<template>
  <div>
    <div id="backbutton-placeholder">
      <router-link to="/calsout"
        ><img src="/static/back_button.png" height="50px" width="50px"
      /></router-link>
    </div>
    
    <form @submit.prevent="input">
      <div id="button-placeholder">
        <br /><b-button type="submit" variant="warning" size="lg" class="mybutton"><b>SUBMIT</b></b-button>
      </div>
      <div id="content-wrap">
        <div id="background"></div>
        <br /><br />

        <div id="date-activity-placeholder">
          <p class="p-3 mb-2 bg-info text-white mytext">Choose a date</p>
          <b-form-datepicker
            id="datepicker"
            menu-class="w-100"
            calendar-width="100%"
            v-model="date"
            :max="maxDate"
            locale="en"
            class="mb-2 mytext"
          ></b-form-datepicker>
          <br />
          <p class="p-3 mb-2 bg-info text-white mytext">Choose a class</p>
          <div>
            <b-form-select
              v-model="class_chosen"
              :options="class_options"
              class="mb-3 mytext"
            >
              <!-- This slot appears above the options from 'options' prop -->
              <template #first>
                <b-form-select-option :value="null" disabled
                  >-- Please select an option --</b-form-select-option
                >
              </template>
            </b-form-select>
          </div>
        </div>
        <br /><br /><br />

        
      </div>
    </form>
    <Footer></Footer>
  </div>
</template>


<script>
import database from "../firebase";
import firebase from "firebase";
import Footer from './Footer.vue'

export default {
  components: {
    Footer
  },
  data() {
    const now = new Date();
    const today = new Date(now.getFullYear(), now.getMonth(), now.getDate());
    const maxDate = new Date(today);

    return {
      maxDate: maxDate,
      class_chosen: null,
      date: null,
      //start: null,
      //end: null,
      space: " ",
      //add_seconds: ':00',
      class_options: [],
      class_list: [],
      unique_classes: [],
    };
  },
  methods: {
    input() {
      if (this.class_chosen == null || this.date == null) {
        // if any field is missing
        alert("Please input all fields!");
      } else {
        var fullDate = new Date(this.date);
        var year = fullDate.getFullYear();
        var month = fullDate.getMonth() + 1;
        var date = fullDate.getDate();
        var day = fullDate.getDay();

        var calories = 0;
        var startHour = 0;
        var endHour = 0;

        var class_chosen = this.class_chosen;

        this.class_list.forEach(function (test) {
          if (class_chosen == test["name"] && day == test["day"]) {
            calories = test["cal"];
            startHour = test["start"].slice(0, 2);
            endHour = test["end"].slice(0, 2);
          }
        });

        if (calories == 0) {
          alert(
            "There is no such class on this day! Please check available classes on our schedule page!"
          );
        } else {
          database
            .collection("inputs")
            .add({
              activity: this.class_chosen,
              calories: Number(calories),
              date: Number(date),
              month: Number(month),
              year: Number(year),
              startHour: Number(startHour),
              endHour: Number(endHour),
              userid: firebase.auth().currentUser.uid,
            })
            .then(() => {
              this.pushToUser(Number(calories));
              alert("Your class is recorded!");
              this.$router.push("/overview");
            })
            .catch((error) => {
              alert(error.message);
            });
        }
      }
    },
    pushToUser: function (calories) {
      var docRef = database
        .collection("user")
        .doc(firebase.auth().currentUser.uid);
      // Atomically increment the burnt calories of user
      docRef.update({
        burnt: firebase.firestore.FieldValue.increment(calories),
      });
    },
    fetchItems: function () {
      database
        .collection("class")
        .get()
        .then((snapshot) => {
          let each_class_option = {};
          let class_list = {};
          snapshot.docs.forEach((doc) => {
            if (this.unique_classes.includes(doc.data()["name"]) == false) {
              each_class_option = {
                value: doc.data()["name"],
                text: doc.data()["name"],
              };
              this.unique_classes.push(doc.data()["name"]);
              this.class_options.push(each_class_option);
            }
            class_list = {
              name: doc.data()["name"],
              cal: doc.data()["cal"],
              start: doc.data()["start"],
              end: doc.data()["end"],
              day: doc.data()["day"],
            };
            this.class_list.push(class_list);
          });
        });
    },
  },
  created() {
    this.fetchItems();
  },
};
</script>

<style scoped>
#background {
  position: absolute;
  background: url(/static/inputactivity_background.png);
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 1;
  z-index: -1;
}
#button-placeholder {
  top: 60%;
  position: absolute;
  z-index: 1;
  text-align: center;
  width: 100%;
}
#content-wrap {
  padding-bottom: 150px;
}
#date-activity-placeholder {
  top: 10%;
  width: 50%;
  left: 25%;
  position: relative;
  z-index: 1;
  text-align: center;
}
#time-placeholder {
  top: 60%;
  width: 100%;
  position: center;
  z-index: 1;
  text-align: center;
}
#backbutton-placeholder {
  top: 13%;
  left: -33%;
  position: absolute;
  z-index: 1;
  text-align: center;
  width: 100%;
}
.mybutton {
  font-family: 'Fjalla One', sans-serif;
}
Footer {
  position: absolute;
  top: 100%;
  width: 100%;
}
.mytext {
  font-family: 'Rubik', sans-serif;
}
</style>