<script setup>
import { ref } from "vue";
import { onMounted } from "vue";

const width = 1600;
const height = (width * 3) / 4;
const linesCount = 10;
const fontSize = 48;
const image = new Image();

const message = ref("I will always follow the DRY principle");
let textColor = ref("#eeeeee");
let bgColor = ref("#365722");
let context = null;

const chalkboardPadding = 20;
const chalkboardWidth = 690;
const chalkboardHeight = 940;

const startAngle = 0.38;
const endAngle = -0.1;
const stepAngle = (startAngle - endAngle) / linesCount;
const stepY = chalkboardHeight / linesCount;

const loadImage = () =>
  new Promise((resolve, reject) => {
    image.src = "./bartfront.png";
    image.onload = () => resolve(image);
    image.onerror = (err) => reject(err);
  });

const loadFont = () =>
  new Promise((resolve, reject) => {
    const font = new FontFace("Neucha", "url(./Neucha-Regular.ttf)");

    font
      .load()
      .then((font) => {
        document.fonts.add(font);
        resolve(font);
      })
      .catch(() => reject(font));
  });

onMounted(() => {
  Promise.all([loadImage(), loadFont()]).then(draw);
  context = document.getElementById("canvas").getContext("2d");
});

function draw() {
  const temp = message.value.split("\n");

  context.fillStyle = bgColor.value;
  context.fillRect(0, 0, canvas.width, canvas.height);
  context.save();
  context.fillStyle = textColor.value;

  for (let i = 0; i < linesCount; i += 1) {
    context.setTransform(1, startAngle - i * stepAngle, 0, 1, 0, 0);
    context.font = `${fontSize}px Neucha`;
    context.fillText(
      temp[i % temp.length],
      chalkboardPadding,
      (i - 2) * stepY + chalkboardPadding,
      chalkboardWidth
    );
  }

  context.fillStyle = bgColor.value;
  context.fillRect(535, 625, canvas.width, canvas.height);
  context.restore();
  context.drawImage(image, 0, 0, canvas.width, canvas.height);
}

function save() {
  const downloadLink = document.createElement("a");
  downloadLink.setAttribute("download", "bart-chalkboard.png");
  document.getElementById("canvas").toBlob(function (blob) {
    downloadLink.setAttribute("href", URL.createObjectURL(blob));
    downloadLink.click();
  });
}
</script>

<template>
  <h1>Bart Chalkboard Generator</h1>
  <hr />
  <div class="wrapper">
    <div class="text-settings">
      <textarea v-model="message" @input="draw" rows="8"></textarea>
    </div>
    <div class="color-settings">
      <div>
        <input type="color" v-model="textColor" @input="draw" /><label
          for="head"
          >Text</label
        >
      </div>

      <div>
        <input type="color" v-model="bgColor" @input="draw" /><label for="body"
          >Background</label
        >
      </div>

      <div><button @click="save">Save high-res PNG</button></div>
    </div>
  </div>

  <canvas :width="width" :height="height" id="canvas"></canvas>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

body {
  background: url("./background.jpg") repeat-y;
  background-size: cover;
}

.wrapper {
  display: flex;
  flex-direction: row;
  justify-content: center;
  margin: 10px 0;
  flex-wrap: wrap;
}

.wrapper > div {
  margin: 0 10px;
}

.color-settings {
  width: 150px;
  text-align: left;
}

.color-settings > div {
  margin: 10px 0;
}

.color-settings label {
  margin-left: 10px;
}

.color-settings button {
  padding: 5px 10px;
  cursor: pointer;
}

.text-settings textarea {
  padding: 10px;
  width: 350px;
}

#canvas {
  width: 500px;
}
</style>
