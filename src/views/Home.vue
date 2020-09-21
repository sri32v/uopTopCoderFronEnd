<template>
  <div class="home">
    <div v-if=false id="nav">
      <top-header></top-header>
    </div>
  <div class="container">
  <div class="row">
        <div class="col-sm-8">Welcome Page</div>
    
  </div>
  
</div>
  </div>
</template>

<script>
// @ is an alias to /src
import topheader from '../components/topheader'
export default {
  name: 'Home',
  components:{"top-header":topheader}
  
}
</script>
