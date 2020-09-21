<template>
  <div v-if="signedIn">
  <b-navbar toggleable="lg" type="dark" variant="danger">
    <b-navbar-brand to="/">
        <span class="font-weight-bold" style="color:white"> Upx </span>
        <span class="code" style="color:white"> TopCoder</span> </b-navbar-brand>

    <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

    <b-collapse id="nav-collapse" is-nav>
      <b-navbar-nav>
      
        <b-nav-item to="/about"><b-button variant="info" size="sm"> About </b-button></b-nav-item>
        <b-nav-item href="/upload"><b-button variant="info"  size="sm">Upload Document</b-button></b-nav-item>
        <b-nav-item href="/docstatus"><b-button variant="info"  size="sm">CheckDoc Status</b-button></b-nav-item>
      </b-navbar-nav>
      <b-navbar-nav>
      
        <b-nav-item to="/profile" ><span class="text-warning">Welcome {{user.username}}</span>  </b-nav-item>
        
      </b-navbar-nav>

      <!-- Right aligned nav items -->
      <b-navbar-nav class="ml-auto">
        <b-nav-form>
          <b-form-input size="sm" class="mr-sm-2" placeholder="Search"></b-form-input>
          <b-button size="sm" class="my-2 my-sm-0" type="submit">Search</b-button>
        </b-nav-form>

        <b-nav-item-dropdown right>
          <!-- Using 'button-content' slot -->
          <template v-slot:button-content>
            <em><b-button variant="info"  size="sm">User</b-button></em>
          </template>
          <b-dropdown-item href="/profile">Profile</b-dropdown-item>
          <b-dropdown-item to="/">
          <span v-if="!signedIn">
        <a v-on:click="signIn">Sign In</a>
      </span>
      <span v-else>
        <a v-on:click="signOut">Sign Out</a>
      </span></b-dropdown-item>
        </b-nav-item-dropdown>
      </b-navbar-nav>
    </b-collapse>
  </b-navbar>
</div>
</template>


<script>
import { Auth } from 'aws-amplify';
export default {
name: "topheader",
data() {
    return {
      signedIn: false,
      user:""
    };
  },
async created() {
    // checks user status on load
    await this.checkUser();
  },
methods: {
    async checkUser() {
      Auth.currentAuthenticatedUser()
      .then((user) => {
       // window.console.log(`user: ${JSON.stringify(user.username)}`);
        this.signedIn = true;
        this.user=user;
      })
      .catch(() => {
        //window.console.log('user not logged in');
        this.signedIn = false;
      });
    },
    async signIn() { // signin button clicked
      await Auth.federatedSignIn({ provider: 'Google' });
    },
    async signOut() {  // signout button clicked

//    console.log("clicked signout")
      await Auth.signOut();
    },
  },
}
</script>

<style>

</style>