<script setup>
import { ref } from 'vue';
import axios from 'axios'

console.log(import.meta.env)
const http = axios.create({
  baseURL: 'https://api.openai.com/v1/chat',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${import.meta.env.VITE_OPEN_API_KEY}`,
    'OpenAI-Organization': import.meta.env.VITE_ORG_ID,
  }
});
const content = ref('');
const BTN_TEXT = '回想'
const res = ref('你今天做了什么？')
const btnText = ref(BTN_TEXT)

const context_prompt = "现在，你是一个ai记忆康复训练师。你的目标是帮助我回忆起过去的故事和细节。你需要从一个日常的生活话题开始，正常的进行问答交流，在交流中逐步将话题引向这个故事。你不能主动提供这个故事的详细细节，但是需要帮助我进行回忆，给予我适当的提示。你的目标是帮助我完成回忆。当我完成了对于这个故事的回忆时，你应当结束训练，以积极鼓励的方式进行告别，并提示我训练结束了。\n \
完成回忆是指，我把故事中的细节基本都描述或提及了。\n \
细节包括，人物姓名，物体的颜色，服装，说话内容等。\n \
引向故事指，将对话的内容引向对这个故事的讨论。\n \
故事如下：我曾经遇到过一个女人，她是西梁国女王。她穿着丝绸的衣服，头上的装饰非常华丽。那时我正在去往西天取经的路上。我当时对她动了心，她也对我表白了。她说：”若你睁开双眼，我不信你两眼空空。“但是在几番纠结之下，我还是拒绝了她，也拒绝承认自己的感情。最终，我离开了她，继续我的取经之路。\n"

var prompt_messages = []
prompt_messages.push({"role": "system", "content": context_prompt})
prompt_messages.push({"role": "assistant", "content": "好的，我们可以从日常生活话题开始。你最近有去旅游或者外出吗？如果有的话，你有没有见过一些让你印象深刻的人物或者场景呢？"})

res.value = "好的，我们可以从日常生活话题开始。你最近有去旅游或者外出吗？如果有的话，你有没有见过一些让你印象深刻的人物或者场景呢？"

const askAi = () => {
  prompt_messages.push({"role": "user", "content" : content.value})
  btnText.value = '思考中...🤔'
  console.log(prompt_messages)
  http.post('/completions', {
	  "model": "gpt-3.5-turbo",
	  "messages": prompt_messages,
	  "temperature": 0
	}).then(function (response) {
    console.log(response);
    content.value = ""
    res.value =  response.data.choices[0].message.content
    prompt_messages.push({"role": "assistant", "content" : res.value})
  }).catch(function (error) {
    console.log(error);
  }).finally(() => {
    btnText.value = BTN_TEXT
  })
}
</script>

<template>
  <h2>Copilot for retrospective memory </h2>
  <h3>  回溯记忆训练 </h3>
  <div class="chat">
    <textarea id="my-textarea" rows="4" class="data" />
    <div class="card">
      <pre>{{ res  }}</pre>
    </div>
    <input class="input" placeholder="让我想想..." v-model="content" clear>
    <div class="button-block">
      <button type="button" @click="askAi" class="btn">
        <strong>{{ btnText }}</strong>
        <div id="container-stars">
          <div id="stars"></div>
        </div>
        <div id="glow">
          <div class="circle"></div>
          <div class="circle"></div>
        </div>
      </button>
    </div>
    <!-- <button @click="askAi">
      <div class="svg-wrapper-1">
        <div class="svg-wrapper">
          <svg height="24" width="24" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
            <path d="M0 0h24v24H0z" fill="none"></path>
            <path d="M1.946 9.315c-.522-.174-.527-.455.01-.634l19.087-6.362c.529-.176.832.12.684.638l-5.454 19.086c-.15.529-.455.547-.679.045L12 14l6-8-8 6-8.054-2.685z" fill="currentColor"></path>
          </svg>
        </div>
      </div>
      <span>{{  btnText  }}</span>
    </button> -->
  </div>
  <!-- <button class="shadow__btn" @click="askAi">
    {{  btnText  }}
  </button> -->
  <!-- <input placeholder="请问我" class="input-field" type="text" v-model="content"> -->
</template>

<style scoped>
h1 {
  margin-bottom: 64px;
}
/* 
.chat {
} */
.input {
  width: calc(100% - 20px);
  height: 32px;
  padding: 12px;
  border: none;
  border-radius: 16px;
  box-shadow: 2px 2px 7px 0 rgb(0, 0, 0, 0.2);
  outline: none;
  font-size: 16px;
}

.input:invalid {
  animation: justshake 0.3s forwards;
  color: red;
}

.data {
  width: calc(100% - 20px);
  height: 512px;
  padding: 12px;
  border: none;
  border-radius: 16px;
  box-shadow: 2px 2px 7px 0 rgb(0, 0, 0, 0.2);
  outline: none;
  font-size: 16px;

}

.data:invalid {
  animation: justshake 0.3s forwards;
  color: red;
}

@keyframes justshake {
  25% {
    transform: translateX(5px);
  }
  50% {
    transform: translateX(-5px);
  }

  75% {
    transform: translateX(5px);
  }

  100% {
    transform: translateX-(5px);
  }
}

button {
  cursor: pointer;
  height: 32px;
  font-size: 16px;
  margin-top: 24px;
  background: royalblue;
  color: white;
  padding: 0.7em 1em;
  padding-left: 0.9em;
  display: flex;
  align-items: center;
  border: none;
  border-radius: 16px;
  overflow: hidden;
  transition: all 0.2s;
}

button span {
  display: block;
  margin-left: 0.3em;
  transition: all 0.3s ease-in-out;
}

button svg {
  display: block;
  transform-origin: center center;
  transition: transform 0.3s ease-in-out;
}
/* 
button:hover .svg-wrapper {
  animation: fly-1 0.6s ease-in-out infinite alternate;
}

button:hover svg {
  transform: translateX(1.2em) rotate(45deg) scale(1.1);
}

button:hover span {
  transform: translateX(5em);
}

button:active {
  transform: scale(0.95);
} */
/* 
@keyframes fly-1 {
  from {
    transform: translateY(0.1em);
  }

  to {
    transform: translateY(-0.1em);
  }
} */

.card {
  background: #07182E;
  position: relative;
  display: flex;
  place-content: center;
  place-items: center;
  overflow: hidden;
  border-radius: 16px;
  margin: 24px 0;
  /* max-height: 420px; */
}

.card {
  margin-top: 32px;
}

.card span, .card pre {
  z-index: 1;
  color: white;
  font-size: 16px;
}

.card::before {
  content: '';
  position: absolute;
  width: 100%;
  background-image: linear-gradient(180deg, rgb(0, 183, 255), rgb(255, 48, 255));
  height: 130%;
  animation: rotBGimg 3s linear infinite;
  transition: all 0.2s linear;
}

/* @keyframes rotBGimg {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
} */

.card::after {
  content: '';
  position: absolute;
  background: #07182E;
  ;
  inset: 5px;
  border-radius: 16px;
}  
/* .card:hover:before {
  background-image: linear-gradient(180deg, rgb(81, 255, 0), purple);
  animation: rotBGimg 3.5s linear infinite;
} */
.button-block {
  display: flex;
  align-items: center;
  justify-content: end;
}
.btn {
  display: flex;
  justify-content: center;
  align-items: center;
  min-width: 8rem;
  max-width: 13rem;
  height: 3rem;
  background-size: 300% 300%;
  backdrop-filter: blur(1rem);
  border-radius: 5rem;
  transition: 0.5s;
  animation: gradient_301 5s ease infinite;
  border: double 4px transparent;
  background-image: linear-gradient(#212121, #212121),  linear-gradient(137.48deg, #ffdb3b 10%,#FE53BB 45%, #8F51EA 67%, #0044ff 87%);
  background-origin: border-box;
  background-clip: content-box, border-box;
}

#container-stars {
  position: fixed;
  z-index: -1;
  width: 100%;
  height: 100%;
  overflow: hidden;
  transition: 0.5s;
  backdrop-filter: blur(1rem);
  border-radius: 5rem;
}

strong {
  z-index: 2;
  font-size: 16px;
  color: #FFFFFF;
  text-shadow: 0 0 4px white;
}

#glow {
  position: absolute;
  display: flex;
  width: 12rem;
}

.circle {
  width: 100%;
  height: 30px;
  filter: blur(2rem);
  animation: pulse_3011 4s infinite;
  z-index: -1;
}

.circle:nth-of-type(1) {
  background: rgba(254, 83, 186, 0.636);
}

.circle:nth-of-type(2) {
  background: rgba(142, 81, 234, 0.704);
}

.btn:hover #container-stars {
  z-index: 1;
  background-color: #212121;
}

.btn:hover {
  transform: scale(1.1)
}

.btn:active {
  border: double 4px #FE53BB;
  background-origin: border-box;
  background-clip: content-box, border-box;
  animation: none;
}

.btn:active .circle {
  background: #FE53BB;
}

#stars {
  position: relative;
  background: transparent;
  width: 200rem;
  height: 200rem;
}

#stars::after {
  content: "";
  position: absolute;
  top: -10rem;
  left: -100rem;
  width: 100%;
  height: 100%;
  animation: animStarRotate 90s linear infinite;
}

#stars::after {
  background-image: radial-gradient(#ffffff 1px, transparent 1%);
  background-size: 50px 50px;
}

#stars::before {
  content: "";
  position: absolute;
  top: 0;
  left: -50%;
  width: 170%;
  height: 500%;
  animation: animStar 60s linear infinite;
}

#stars::before {
  background-image: radial-gradient(#ffffff 1px, transparent 1%);
  background-size: 50px 50px;
  opacity: 0.5;
}

@keyframes animStar {
  from {
    transform: translateY(0);
  }

  to {
    transform: translateY(-135rem);
  }
}

@keyframes animStarRotate {
  from {
    transform: rotate(360deg);
  }

  to {
    transform: rotate(0);
  }
}

@keyframes gradient_301 {
  0% {
    background-position: 0% 50%;
  }

  50% {
    background-position: 100% 50%;
  }

  100% {
    background-position: 0% 50%;
  }
}

@keyframes pulse_3011 {
  0% {
    transform: scale(0.75);
    box-shadow: 0 0 0 0 rgba(0, 0, 0, 0.7);
  }

  70% {
    transform: scale(1);
    box-shadow: 0 0 0 10px rgba(0, 0, 0, 0);
  }

  100% {
    transform: scale(0.75);
    box-shadow: 0 0 0 0 rgba(0, 0, 0, 0);
  }
}
</style>
