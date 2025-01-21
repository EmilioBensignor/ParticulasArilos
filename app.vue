<template>
  <div class="page-container">
    <div class="canvas-container">
      <h1>{{ title }}</h1>
      <p>{{ subtitle }}</p>
      <button>{{ buttonText }}</button>
    </div>
    <PomegranateCanvas :maxWidth="1200" />
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
.canvas-container {
  text-align: center;
  padding: 2rem;
  background: white;
}

h1 {
  margin-bottom: 1rem;
  font-size: 2rem;
  font-weight: bold;
}

p {
  padding: 0 2rem;
  margin-bottom: 2rem;
}

button {
  margin-top: 2rem;
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  background-color: #bc2026;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #a51920;
}
</style>