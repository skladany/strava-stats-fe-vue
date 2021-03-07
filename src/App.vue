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
const capitalize = (s) => {
  if (typeof s !== "string") return "";
  return s.charAt(0).toUpperCase() + s.slice(1);
};

export default {
  name: "App",
  data: function() {
    return {};
  },
  computed: {
    ATHLETE: function() {
      return window.location.hostname.includes("ashley") ? "ashley" : "steve";
    },
    headline: function() {
      const athlete = capitalize(this.ATHLETE);

      return "steve" === this.ATHLETE
        ? `👟 ${athlete} Runs the World! 🏃‍♂️`
        : `🦄 ${athlete} Runs the World! 🏃‍♀️`;
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
