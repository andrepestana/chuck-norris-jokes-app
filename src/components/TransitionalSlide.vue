<template>
  <div id="transitional-slide-div">
    
  </div>
</template>

<script>
export default {
  mounted() {
    document.querySelector('#transitional-slide-div').style.height = window.innerHeight + 'px'
  }
}
</script>

<style>
#transitional-slide-div {
  /* The image used */
  background-image: url("~assets/explosion.jpg");

  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
  background-position: center; /* Center the image */
  background-repeat: no-repeat; /* Do not repeat the image */
  z-index: 99;
}

</style>