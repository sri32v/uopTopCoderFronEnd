<template>
  <div class="profile container">
    <h3>Logged in User Cognito Profile page</h3>
    
    <div class="alert-success">User id: {{username}}</div>
    <!-- <div class="alert-success">UserPool id: {{user.pool}}</div> -->
       <div class="alert-success">UserEmail: {{email}}</div>
  </div>
</template>

<script>
import { Auth } from 'aws-amplify';
export default {
data() {
    return {
      signedIn: false,
      user:"",
      username:"",
      email:""
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
        //window.console.log(`user: ${JSON.stringify(user)}`);
        this.signedIn = true;
        this.user=user;
        this.username=user.username;
        this.email=user.attributes.email;
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

      await Auth.signOut();
    },
  },
}
</script>

<style>

</style>
