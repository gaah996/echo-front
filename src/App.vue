<template>
  <div id="app">
    <div>
      Ouvindo o canal:
      <input type="text" v-model="channel" @input="listen()" />
    </div>
    <img alt="Vue logo" src="./assets/logo.png" />
    <div class="input">
      Mensagem:
      <textarea type="text" v-model="input"></textarea>
      <button @click="request()">Enviar</button>
    </div>
    <hr />
    <span :key="index" v-for="(item, index) in list">{{item}}</span>
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";
import Echo from "laravel-echo";
import axios from "axios";

export default {
  name: "app",
  components: {
    HelloWorld
  },
  data: () => ({
    list: [],
    input: "",
    channel: ""
  }),
  created() {
    this.randomChannel();
  },
  methods: {
    async request() {
      try {
        const result = await axios.get(
          `http://localhost/test-broadcast?message=${this.input}&channel=${this.channel}`
        );
        this.input = "";
      } catch (err) {
        console.error(err.data);
      }
    },
    randomChannel() {
      this.channel = Math.random()
        .toString(36)
        .substring(6);
      this.listen();
    },
    listen() {
      window.io = require("socket.io-client");

      window.Echo = new Echo({
        broadcaster: "socket.io",
        host: window.location.hostname + ":6001"
      });

      window.Echo.leave("laravel_database_test-" + this.channel);
      this.list = [];

      window.Echo.channel("laravel_database_test-" + this.channel).listen(
        ".test.event",
        e => {
          this.list.push(e.data);
        }
      );
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0 auto;
  max-width: 500px;
}
#app img {
  width: 60px;
  margin: 10px;
}
.input {
  display: flex;
  flex-direction: column;
  padding: 0 50px;
}
.input textarea {
  height: 150px;
  margin: 20px 0;
}
.input button {
  width: fit-content;
  align-self: flex-end;
}
hr {
  margin: 20px 0 0;
}
span {
  padding: 20px;
  display: block;
  border-bottom: 1px solid #cecece;
}
span:last-child {
  border-bottom: none;
}
</style>
