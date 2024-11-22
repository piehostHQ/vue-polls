<template>
  <div class="">
    <div class="poll-container ">
      <h2 class="poll-question">{{ question }}</h2>
      <ul class="poll-options">
        <li v-for="option in options" :key="option.value" class="poll-option">
          {{ option.text }} (Votes: {{ option.votes }})
          <button @click="vote(option)" class="vote-button">Vote</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, reactive } from 'vue';
import PieSocket from "piesocket-js";

const question = ref("What's your favorite programming language?");
const options = ref([
  { value: 1, text: 'Vue', votes: 0 },
  { value: 2, text: 'React', votes: 0 },
  { value: 3, text: 'Laravel', votes: 0 },
  { value: 4, text: 'JS', votes: 0 },
]);

const vote = (option) => {
  option.votes++;
  broadcastVote(option);
};

let broadcastVote;
const pieSocket = new PieSocket({
    clusterId: "s12117.nyc1",
    apiKey: "0jlifmULE2eJFtCI2kjabsR7EdgaY4EWRZgdGMkE",
});

pieSocket.subscribe("vote-room").then(ch => {
  const channel = ch;

  channel.listen("vote", (data) => {
    const { value } = JSON.parse(data);
    options.value.find(option => option.value === value).votes++;
  }); 

  broadcastVote = (option) => {
    channel.publish("vote", JSON.stringify({ value: option.value }));
  };
}).catch(error => {
  console.error("Error subscribing to PieSocket channel:", error);
});
</script>

<style scoped>
.poll-container {
  font-family: Arial, sans-serif;
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.poll-question {
  font-size: 20px;
  margin-bottom: 10px;
  color: black;
}

.poll-options {
  list-style: none;
  padding: 0;
  color: black;
}

.poll-option {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  color: black;
}

.vote-button {
  background-color: #007bff;
  color: #fff;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.vote-button:hover {
  background-color: #0056b3;
}
</style>