<template>
  <div id="alert">
    <div class="shadow"></div>
    <div class="content">
      <h1 class="animated tada">Par!</h1>
      <h2>'Nothing lost, nothing gained.'</h2>
      <button class="btn btn-lg btn-light mt-3">Replay</button><br/>
      <button class="btn btn-lg btn-light mt-3">Next Hole</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Alert',
  props: {

  }
}
</script>
<style scoped>
#alert {
  width:100vw;
  height:100vh;
  position:absolute;
  top:0;
  left:0;
  z-index:9999;
}

#alert .shadow {
  width:100vw;
  height:100vh;
  position:absolute;
  top:0;
  left:0;
  z-index:7777;
  background:rgba(0,0,0,0.5);
}

#alert .content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align:center;
  z-index:8888;
}
#alert .content h1 {
  font-family: 'Luckiest Guy', cursive !important;
  font-weight:bold;
  text-shadow: -1px 1px 2px #000,
          1px 1px 2px #000,
          1px -1px 0 #000,
          -1px -1px 0 #000;
  font-size:8em;
  color:#FFF;
  margin:0;
  margin-bottom:-20px;
  padding:0;
}
#alert .content h2 {
  font-family: 'Lalezar', cursive;
  font-size:1.8em;
  text-shadow: -1px 1px 2px #000,
          1px 1px 2px #000,
          1px -1px 0 #000,
          -1px -1px 0 #000;
  color:#FFF;
  margin:0;
  padding:0;
}
#alert .content button {
  font-family: 'Luckiest Guy', cursive !important;
  border:4px solid black;
}
</style>