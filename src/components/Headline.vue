<template>
  <div id="header-container">
    <h1>Dance of terror in Kashmir</h1>
    <p>
      2020 witnessed an improved security situation in Jammu and Kashmir.
      However, any picture of normalcy was not acceptable to the terrorists.
    </p>
  </div>
</template>

<script>
export default {
  name: "Headline"
};
</script>

<style lang="scss">
$red: rgb(250, 125, 125);
#header-container {
  width: 100vw;
  height: 100vh;
  text-align: center;
  color: white;

  h1 {
    font-family: "Playfair Display", serif;
    display: block;
    margin: 150px auto 0 auto;
    font-size: 6rem;
    max-width: 800px;
    color: $red;
  }
  p {
    display: block;
    margin: 10px auto 0 auto;
    max-width: 900px;
    font-size: 1.2rem;
  }
}

@media (max-width: 500px) {
  #header-container {
    max-width: 300px;
    margin: auto;
    h1 {
      font-size: 4rem;
      margin-bottom: 3rem;
    }
    p {
      font-size: 1rem;
    }
  }
}
</style>
