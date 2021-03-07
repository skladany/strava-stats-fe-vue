<template>
  <div id="app">
    <h1>{{ headline }}</h1>
    <div class="progress-bar">
      <div class="bar" style=""></div>
      <p>
        <strong>{currentMiles}</strong> of <strong>{mileageGoal}</strong> 🏁
        miles.
      </p>
    </div>
    <h2 id="goal-status" class="{mileageGoalComplete}">
      🎉 You did it!!! 🎉
    </h2>
    <p>
      You are <strong>{currPace()}</strong> miles off your yearly goal pace of
      <strong>{goalPace()}</strong>.
    </p>
    <p>You have <strong>{daysLeft()}</strong> days left to hit your goal!</p>
    <p>Run <strong>{milesPerDay()}</strong> miles a day to hit your goal!</p>
    <p>&hellip;</p>
    <p>
      <strong>{currentWeeklyMiles}</strong> of
      <strong>{weeklyMileageGoal}</strong> weekly miles.
      <br />
      <strong>
        {parseFloat(weeklyMileageGoal - currentWeeklyMiles).toFixed(2)}
      </strong>
      to go!!!
    </p>
    <div className="toggles">
      <button>👇</button>
      <button>👆</button>
    </div>
  </div>
</template>

<script>
const ATHLETE = window.location.hostname.includes("ashley")
  ? "ashley"
  : "steve";

const ENDPOINT = `https://api-postal-run.herokuapp.com`;
//const ENDPOINT = `http://localhost:7777`;

const capitalize = (s) => {
  if (typeof s !== "string") return "";
  return s.charAt(0).toUpperCase() + s.slice(1);
};

export default {
  name: "App",
  data: function() {
    return {
      runData: [],
    };
  },
  created: function() {
    this.fetchData();
  },
  computed: {
    headline: function() {
      const athlete = capitalize(ATHLETE);

      return "steve" === ATHLETE
        ? `👟 ${athlete} Runs the World! 🏃‍♂️`
        : `🦄 ${athlete} Runs the World! 🏃‍♀️`;
    },
  },
  methods: {
    fetchData: async function() {
      try {
        let page = 1;

        if (process.env.NODE_ENV === "_development") {
          //runData = testData;
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
          console.log(this.runData);
        }
      } catch (e) {
        console.log(e);
      }
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
