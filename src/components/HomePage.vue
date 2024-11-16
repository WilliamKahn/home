/* eslint-disable vue/multi-word-component-names */
<template>
  <div class="container">
    <div class="content">
      <h1>{{ name }}</h1>
      <p>{{ description }}</p>
      <div class="buttons">
        <button v-for="button in buttons" :key="button.text" @click="button.action">
          {{ button.text }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HomePage',
  data() {
    return {
      name: "William Kahn",
      description: "A brief introduction about yourself.",
      buttons: [
        { text: "Button 1", action: this.button1Action },
        { text: "Button 2", action: this.button2Action },
        { text: "Button 3", action: this.button3Action },
      ],
    };
  },
  methods: {
    button1Action() {
      alert("Button 1 clicked");
    },
    button2Action() {
      alert("Button 2 clicked");
    },
    button3Action() {
      alert("Button 3 clicked");
    },
    setRandomBackground() {
      const colors = [
        "#ff9a9e", "#fad0c4", "#fad0c4", "#fbc2eb", "#a18cd1", "#fbc2eb",
        "#ffecd2", "#fcb69f", "#ff9a9e", "#ffdde1", "#c3cfe2", "#c3cfe2"
      ];
      const randomColor1 = colors[Math.floor(Math.random() * colors.length)];
      const randomColor2 = colors[Math.floor(Math.random() * colors.length)];
      document.body.style.background = `linear-gradient(to right, ${randomColor1}, ${randomColor2})`;
    },
  },
  mounted() {
    this.setRandomBackground();
  },
};
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
  overflow: hidden; /* Disable scrolling */
}
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%; /* Match parent height */
  text-align: center;
}
.content {
  /* Removed the white background */
  padding: 20px;
  border-radius: 10px;
}
h1 {
  background: linear-gradient(to right, #ff9a9e, #fad0c4);
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
  font-size: 3em; /* Increased font size */
}
.buttons {
  margin-top: 20px;
}
button {
  margin: 5px;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>