<template>
  <div class="docStatus">
    <h4>This is an DocStatus page</h4>
    <!-- <div class="">{{docData}}</div> -->
    <div class="container-fluid">
      <table class="table table-bordered">
        <thead class="table-primary">
          <tr>
            <th scope="col">#</th>
            <th scope="col">PersonId</th>
            <th scope="col">Email</th>
            <th scope="col">FirstName</th>
            <th scope="col">LastName</th>
            <th scope="col">Age</th>
            <th scope="col">Status</th>
            <th scope="col">DocId</th>
            <th scope="col">DocCode</th>
            <th scope="col">S3Url</th>
          </tr>
        </thead>
        <tbody>
          <tr
            class="table-info"
            style="padding-bottom: 6em;"
            v-for="(item, index) in docData"
            :key="item"
          >
            <th scope="row">{{ index + 1 }}</th>
            <td>{{ item.PersonId }}</td>
            <td>{{ item.email }}</td>
            <td>{{ item.FirstName }}</td>
            <td>{{ item.LastName }}</td>
            <td>{{ item.age }}</td>
            <td>{{ item.DocStatus }}</td>
            <td>{{ item.DocId }}</td>
            <td>{{ item.DocCode }}</td>
            <td>
              <a v-bind:href="item.S3Url">{{ item.S3Url }}</a>
            </td>
          </tr>
         </tbody>
      </table>
      <!-- <div class="" v-for="(item, index) in docData" :key="item">{{item.S3Url}}{{index}}</div> -->
      </div>
  </div>
</template>
<script>
import axios from "axios";
import { Auth } from "aws-amplify";
export default {
  data() {
    return {
      signedIn: false,
      user: "",
      docData: [],
    };
  },
  async created() {
    // checks user status on load
    await this.getUser();
  },
  methods: {
    async getUser() {
      try {
        let user = await Auth.currentAuthenticatedUser();
        const docGetUrl =
          "https://api.upxtopcoder.com/V2/personid/" + user.username;
        let docDataGet = await axios.get(docGetUrl);

        //this.docData=docData.items;
        console.log(docDataGet.data.Items);
        this.docData = docDataGet.data.Items;
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>
