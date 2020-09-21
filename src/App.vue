<template>
  <div id="app">
    <div v-if="signedIn" id="nav">
      <top-header></top-header>
    </div>
    <router-view/>
  </div>
</template>
<script>
import topheader from './components/topheader'
import { AmplifyEventBus } from "aws-amplify-vue";
import { Auth } from "aws-amplify";

export default {
  components:{"top-header":topheader},
  data(){
    return {
signedIn:false
    }
  },
  created(){

this.findUser();
 AmplifyEventBus.$on('authState',info=>{
  if (info==='signedIn'){
    this.findUser();
  } else{
    this.signedIn=false;
  }
})
  },
  methods:{
   async findUser() {
      try {
        //const user = await Auth.currentAuthenticatedUser();
         await Auth.currentAuthenticatedUser();
        this.signedIn=true;
        //console.log(user);
      } catch (err) {
        this.signedIn=false;
        
      }
    }
  }
}
</script>
<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
