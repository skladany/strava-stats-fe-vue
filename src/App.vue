<template>
  <div id="app">
    <h1>{{ headline }}</h1>
    <div v-if="isLoading">
      <img src="/loading.gif" />
    </div>
    <div v-else>
      <div class="progress-bar">
        <div class="bar" :style="{ width: progressBar }"></div>
        <p>
          <strong>{{ currentMiles }}</strong> of
          <strong>{{ mileageGoal }}</strong> 🏁 miles.
        </p>
      </div>
      <h2 id="goal-status" class="{mileageGoalComplete}">
        🎉 You did it!!! 🎉
      </h2>
      <p>
        You are <strong>{{ currPace }}</strong> miles off your yearly goal pace
        of <strong>{{ goalPace }}</strong
        >.
      </p>
      <p>
        You have <strong>{{ daysLeft }}</strong> days left to hit your goal!
      </p>
      <p>
        Run <strong>{{ milesPerDay }}</strong> miles a day to hit your goal!
      </p>
      <p>&hellip;</p>
      <p>
        <strong>{{ currentWeeklyMiles }}</strong> of
        <strong>{{ weeklyMileageGoal }}</strong> weekly miles.
        <br />
        <strong>
          {{ weeklyMilesLeft }}
        </strong>
        to go!!!
      </p>
      <div class="toggles">
        <button @click="toggleMileage(-1)">👇</button>
        <button @click="toggleMileage(1)">👆</button>
      </div>
    </div>
  </div>
</template>

<script>
import testData from "./data/testData";

const ATHLETE = window.location.hostname.includes("ashley")
  ? "ashley"
  : "steve";

const ENDPOINT = `https://api-postal-run.herokuapp.com`;
//const ENDPOINT = `http://localhost:7777`;

const capitalize = (s) => {
  if (typeof s !== "string") return "";
  return s.charAt(0).toUpperCase() + s.slice(1);
};

const metersToMiles = (meters) => {
  return (meters * 0.000621371).toFixed(2);
};

const currDayNumber = () => {
  const today = new Date();
  const day = Math.ceil(
    (today - new Date(today.getFullYear(), 0, 1)) / 86400000
  );

  return day;
};

// Get the start of the week, given the current date
// https://www.w3resource.com/javascript-exercises/javascript-date-exercise-50.php
const startOfWeek = (date) => {
  const day = date.getDate() - date.getDay() + (date.getDay() === 0 ? -6 : 1);

  date.setDate(day);
  date.setHours(0, 0, 0, 0);

  return new Date(date);
};

export default {
  name: "App",
  data: function() {
    return {
      runData: [],
      isLoading: true,
      progressBarWidth: "0%",
      weeklyMileageGoal: 0,
    };
  },
  created: function() {
    this.fetchData();
    this.getWeeklyMileageGoal();
    setTimeout(() => {
      this.setProgressBarWidth();
    }, 1000);
  },
  computed: {
    headline: function() {
      const athlete = capitalize(ATHLETE);

      return "steve" === ATHLETE
        ? `👟 ${athlete} Runs the World! 🏃‍♂️`
        : `🦄 ${athlete} Runs the World! 🏃‍♀️`;
    },
    currentMiles: function() {
      const distance = this.runData.reduce((totalDistance, { distance }) => {
        return totalDistance + distance;
      }, 0);

      return metersToMiles(distance);
    },
    mileageGoal: function() {
      return 200;
    },
    daysLeft: function() {
      const today = new Date();
      const day = Math.floor(
        (new Date(today.getFullYear() + 1, 0, 1) - today) / 86400000
      );

      return day;
    },
    goalPace: function() {
      // Ideal Pace
      const milesPerDay = this.mileageGoal / (currDayNumber() + this.daysLeft);
      const goalPace = currDayNumber() * milesPerDay;

      return goalPace.toFixed(2);
    },
    currPace: function() {
      return (this.goalPace - this.currentMiles).toFixed(2);
    },
    milesPerDay: function() {
      return parseFloat(
        (this.mileageGoal - this.currentMiles) / this.daysLeft
      ).toFixed(2);
    },
    currentWeeklyMiles: function() {
      const today = new Date();
      const monday = startOfWeek(today);

      const thisWeek = this.runData
        .filter(({ start_date }) => {
          // @todo this is UTC time, need to account for timezones eventually
          // ...but generally close enough for now
          const activityDate = new Date(start_date);

          return activityDate > monday;
        })
        .reduce((totalDistance, { distance }) => {
          return totalDistance + distance;
        }, 0);

      return metersToMiles(thisWeek);
    },
    weeklyMilesLeft: function() {
      return parseFloat(
        this.weeklyMileageGoal - this.currentWeeklyMiles
      ).toFixed(2);
    },
    progressBar: function() {
      return this.progressBarWidth;
    },
  },
  methods: {
    fetchData: async function() {
      try {
        let page = 1;

        if (process.env.NODE_ENV === "development") {
          this.runData = testData;
        } else {
          /* eslint-disable no-constant-condition */
          while (true) {
            const data = await fetch(
              `${ENDPOINT}/strava/distance_run/${ATHLETE}/?page=${page}&per_page=50`
            ).then((r) => r.json());

            if (data.length < 1) {
              break;
            }

            this.runData = [...this.runData, ...data];

            page++;
          } // end while
          /* eslint-enable no-constant-condition */
        }
        this.isLoading = false;
      } catch (e) {
        console.log(e);
      }
    },
    getWeeklyMileageGoal: function() {
      const weeklyGoal = parseInt(localStorage.getItem("weeklyGoal")) || 40;
      this.weeklyMileageGoal = weeklyGoal;
    },
    toggleMileage: function(mile) {
      const weeklyGoal =
        this.weeklyMileageGoal + mile > 0 ? this.weeklyMileageGoal + mile : 0;
      this.weeklyMileageGoal = weeklyGoal;
      localStorage.setItem("weeklyGoal", weeklyGoal);
    },
    setProgressBarWidth: function() {
      this.progressBarWidth = `40%`;
    },
  },
};
</script>

<style></style>
