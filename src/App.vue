<template>
  <div id="app">
    <div>
      <label for="min_time">Min Time</label>
      <input ref="min" type="text" maxlength="6" size="8" name="min_time" id="min_time" placeholder="MM:SS" :disabled="running" @input="minInput">
    </div>
    <div>
      <label for="max_time">Max Time</label>
      <input ref="max" type="text" maxLength="6" size="8" name="max_time" id="max_time" placeholder="MM:SS" :disabled="running" @input="maxInput">
    </div>
    <div>
      <button ref="btn" id="btn" @click="click" v-text="running ? 'Stop' : 'Start'" :disabled="!valid">Start</button>
      <div v-show="running">
        <label for="cur_time">Remaining Time</label>
        <div ref="cur">{{ remainingStr }}</div>
      </div>
    </div>
  </div>
</template>

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

<script>

var timeRE = /\d+:?(\d\d?)?/;

var alarm = new Audio('alarm.wav');

function randomRange(min, max) {
  return Math.floor((Math.random() * (max - min)) + min);
}

export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      running: false,
      min: 0, //seconds
      max: 0, //seconds
      timer: null,
      cur: 0, // milliseconds
      start: 0,
      now: 0,
    }
  },
  computed: {
    remaining() {
      return this.cur - (this.now - this.start)
    },
    remainingStr() {

      if (this.remaining < 0) {
        return '0:00';
      }

      var m = Math.floor(this.remaining / 60000); //milliseconds to minutes
      var s = Math.floor(this.remaining % 60000 / 1000); //milliseconds to remainder seconds

      var str = m + ':';

      if (s < 10) {
        str += '0';
      }

      str += s;

      return str;
    },
    valid() {
      return (this.max > 0 && this.max > this.min);
    }
  },
  methods: {
    minInput() {
      var val = this.$refs.min.value;

      console.log('minInput: ' + val);
      if (val == ':') {
        val = '0:';
      }

      val = String.prototype.match.call(val, timeRE)[0];
      if (!val) {
        val = '';
      }

      this.$refs.min.value = val;
      var sp = val.split(':');
      this.min = parseInt(sp[0]) * 60 + (sp[1] ? parseInt(sp[1]) : 0);
    },

    maxInput() {
      var val = this.$refs.max.value;

      console.log('maxInput: ' + val);
      if (val == ':') {
        val = '0:';
      }

      val = String.prototype.match.call(val, timeRE)[0];
      if (!val) {
        val = '';
      }

      this.$refs.max.value = val;
      var sp = val.split(':');
      var sec = parseInt(sp[0]) * 60 + (sp[1] ? parseInt(sp[1]) : 0);
      this.max = sec;
    },

    click() {
      this.running = !this.running;

      if (this.running) {
        //start
        this.cur = randomRange(this.min, this.max) * 1000;
        this.start = Date.now();
        this.now = this.start;

        this.timer = setInterval(this.update.bind(this), 500);
      } else {
        //stop
        this.cur = 0;
        this.start = 0;
        this.now = 0;
        clearInterval(this.timer);
        this.timer = null;
      }
    },

    update() {
      this.now = Date.now();
      if (this.remaining < 0) {
        alarm.play();
      }
    }
  }
}
</script>
