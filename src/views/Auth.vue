<template>
  <div class="auth-page">
    <amplify-authenticator></amplify-authenticator>
  </div>
</template>

<script>
import { AmplifyEventBus } from "aws-amplify-vue";
import { Auth } from "aws-amplify";

export default {
  async mounted() {
    AmplifyEventBus.$on("authState", (eventInfo) => {
      if (eventInfo === "signedIn") {
        this.$router.push({ name: "Home" });
      } else if (eventInfo === "signedOut") {
        this.$router.push({ name: "Auth" });
      }
    });
    try {
      await Auth.currentAuthenticatedUser();
      this.$router.push({ name: "Home" });
    } catch (error) {
      this.$router.push({ name: "Auth" });
    }
  },
};
</script>

<style scoped>
.auth-page {
  height: 100vh;
  padding-top: 25vh;
  background-color: lightgray;
}
</style>
