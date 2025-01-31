<template>
  <div class="page-container">
    <section class="canvasSection">
      <div class="hero">
        <h1>{{ title }}</h1>
        <p>{{ subtitle }}</p>
        <div class="buttonContainer">
          <button>{{ buttonText }}</button>
        </div>
      </div>
      <!-- <PomegranateCanvas class="arilosCanvas" :maxWidth="1200" /> -->
      <ArilosCanvas class="arilosCanvas" :maxWidth="1200" />
    </section>

    <div>
      <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Rem autem vero amet. Fugiat blanditiis illo repellat
        voluptate, tempora ut omnis impedit? Similique voluptates a accusamus harum, ab velit ipsum dolores?</p>
      <div class="canvas-container">
        <h1>{{ title }}</h1>
        <p>{{ subtitle }}</p>
        <button>{{ buttonText }}</button>
      </div>
      <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Rem autem vero amet. Fugiat blanditiis illo repellat
        voluptate, tempora ut omnis impedit? Similique voluptates a accusamus harum, ab velit ipsum dolores?</p>
      <div class="canvas-container">
        <h1>{{ title }}</h1>
        <p>{{ subtitle }}</p>
        <button>{{ buttonText }}</button>
      </div>
      <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Rem autem vero amet. Fugiat blanditiis illo repellat
        voluptate, tempora ut omnis impedit? Similique voluptates a accusamus harum, ab velit ipsum dolores?</p>
      <div class="canvas-container">
        <h1>{{ title }}</h1>
        <p>{{ subtitle }}</p>
        <button>{{ buttonText }}</button>
      </div>
      <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Rem autem vero amet. Fugiat blanditiis illo repellat
        voluptate, tempora ut omnis impedit? Similique voluptates a accusamus harum, ab velit ipsum dolores?</p>
      <div class="canvas-container">
        <h1>{{ title }}</h1>
        <p>{{ subtitle }}</p>
        <button>{{ buttonText }}</button>
      </div>
      <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Rem autem vero amet. Fugiat blanditiis illo repellat
        voluptate, tempora ut omnis impedit? Similique voluptates a accusamus harum, ab velit ipsum dolores?</p>
      <div class="canvas-container">
        <h1>{{ title }}</h1>
        <p>{{ subtitle }}</p>
        <button>{{ buttonText }}</button>
      </div>

    </div>
  </div>
</template>

<script setup>
const title = 'Calidad. Frescura. Consistencia.'
const subtitle = 'Granadas seleccionadas con entregas rápidas y consistentes.'
const buttonText = 'Hace tu pedido'
</script>

<style>
body {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}
</style>

<style scoped>
.page-container {
  min-height: 100vh;
}

.canvasSection {
  position: relative;
  height: 50vh;
}

/* .canvasSection>h1,
.canvasSection>p,
.canvasSection>.buttonContainer {
  position: absolute;
  left: 0;
  right: 0;
  z-index: 2;
  margin: 0;
  width: max-content;
  margin: 0 auto;
} */

.hero {
  position: absolute;
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 2;
  border-radius: 999px;
  padding: 1rem;
  border-radius: 8px;
  max-width: 600px;
  text-align: center;
  margin: 0 auto;
  left: 0;
  top: 8rem;
  right: 0;
  /* box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); */
}

.canvasSection h1 {
  font-size: 2.5rem;
  font-weight: bold;
  color: #2D3748;
  margin: 1rem 0;
}

.canvasSection p {
  color: #4A5568;
}

.canvasSection button {
  margin-top: 1rem;
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  background-color: #bc2026;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.hero button:hover {
  background-color: #a51920;
}

.arilosCanvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
}
</style>