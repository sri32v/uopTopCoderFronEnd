<template>
  <div id="fileUpload">
    <h1>Upload Document</h1>

    <div class="container" v-if="error">
      <b-alert show variant="danger" dismissible>
        {{ message }}
      </b-alert>
    </div>

    <div v-if="!UploadSuccess" class="fileUpload-form" align="left">
      <form @submit.prevent="saveData" enctype="multipart/form-data">
        <div class="input">
          <label for="email">Mail</label>
          <input type="email" id="email" v-model="email" />
        </div>

        <div class="input">
          <label for="fName">First Name</label>
          <input type="text" id="fName" v-model="fName" />
        </div>
        <div class="input">
          <label for="lName">LastName</label>
          <input type="lName" id="lName" v-model="lName" />
        </div>
        <div class="input">
          <label for="Title">Title</label>
          <input type="Title" id="Title" v-model="Title" />
        </div>
        <div class="input">
          <label for="age">Age</label>
          <input type="number" id="age" v-model.number="age" />
        </div>
        <div class="input">
          <label for="doccode">Document Code</label>
          <select id="doccode" v-model="doccode">
            <option value="UOPUS_Transcript">UOPUS_Transcript</option>
            <option value="UOPUS_DrivingLicense">UOPUS_DrivingLicense</option>
          </select>
        </div>
        <div class="input">
          <label for="doctype">Document Type</label>
          <select id="doctype" v-model="doctype">
            <option value="PDF">PDF</option>
            <option value="JPEG">JPEG</option>
          </select>
        </div>
        <div class="comments" v-if="!hide">
          <button @click="onAddComment" type="button">Add Comments</button>
          <div class="comment-list">
            <div
              class="input"
              v-for="(commentInput, index) in commentInputs"
              :key="commentInput.id"
            >
              <label :for="commentInput.id">Comments #{{ index + 1 }}</label>
              <input
                type="text"
                :id="commentInput.id"
                v-model="commentInput.value"
              />
              <button @click="onDeleteComment(commentInput.id)" type="button">
                X
              </button>
            </div>
          </div>
        </div>
        <div class="input inline" v-if="!hide">
          <input type="checkbox" id="terms" v-model="terms" />
          <label for="terms">Accept Terms of Use</label>
        </div>
        <div class="dropzone">
          <input
            type="file"
            class="input-file"
            ref="file"
            @change="selectFile"
          />
          <p class="call-to-action">Drag files here</p>
        </div>

        <div v-if="fexists" class="message alert alert-success">
          {{ file.name }} <br />Size:{{ file.size / 1000 }}kb
        </div>

        <div class="submit">
          <button type="submit">Submit</button>
        </div>
      </form>
    </div>
    <div v-if="UploadSuccess" class="container alert alert-success">
      Document submitted Successfully <br />
      ConfirmationNumber: {{ confirmNum }}
      <div class="">
        Image Url:
        <a v-bind:href="S3Url">{{ S3Url }}</a>
      </div>
      <b-nav-item href="/upload"
        ><b-button variant="info" size="md"
          >Upload Another document</b-button
        ></b-nav-item
      >
    </div>
  </div>
</template>

<script>
//import dropzone from './components/dropzone.vue'
//import _ from "lodash";
import axios from "axios";
import { Auth } from "aws-amplify";
export default {
  name: "Upload",
  data() {
    return {
      email: "",
      commentInputs: [],
      uploadFiles: [],
      files: [],
      message: "",
      error: false,
      file: "",
      fexists: false,
      fName: "",
      lName: "",
      age: "",
      doccode: "",
      doctype: "",
      terms: false,
      hide: true,
      confirmNum: "",
      S3Url: "",
      user: "",
      Title: "",
      UploadSuccess: false,
    };
  },
  // components:{dropzone},
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
          this.user = user;
        })
        .catch(() => {
          //window.console.log('user not logged in');
          this.signedIn = false;
        });
    },
    onAddComment() {
      const newcomment = {
        id: Math.random() * Math.random() * 1000,
        value: "",
      };
      this.commentInputs.push(newcomment);
    },
    onDeleteComment(id) {
      this.commentInputs = this.commentInputs.filter(
        (comment) => comment.id !== id
      );
    },
    selectFile() {
      // const files = this.$refs.files.files;
      // console.log(files);
      // this.uploadFiles = [...this.uploadFiles, ...files];
      // this.files = [
      //   ...this.files,
      //   ..._.map(files, (file) => ({
      //     name: file.name,
      //     size: file.size,
      //     type: file.type,
      //     invalidMessage: this.validate(file),
      //   })),
      // ];
      const file = this.$refs.file.files[0];
      const allowedTypes = [
        "image/jpeg",
        "image/png",
        "image/gif",
        "application/pdf",
      ];
      const MAX_SIZE = 10000000;
     // console.log(file);
      const tooLarge = file.size > MAX_SIZE;
      if (allowedTypes.includes(file.type) && !tooLarge) {
        this.file = file;
        this.error = false;
        this.fexists = true;
        this.message = "";
      } else {
        this.error = true;
        this.message = tooLarge
          ? `Too large. Max size is ${MAX_SIZE / 1000/1000}MB`
          : "Only images,Pdfs are allowed";
      }
    },
    async saveData() {
      const formData = new FormData();

      let documentCode = "";
      formData.append("file", this.file);
      switch (this.doccode) {
        case "UOPUS_DrivingLicense":
          documentCode = "DR";
          break;
        case "UOPUS_Transcript":
          documentCode = "TR";
          break;
        default:
          documentCode = "OTH";
      }

      const UploadUrl =
        "https://api.upxtopcoder.com/V1/fileUpload?type=" +
        documentCode +
        "&pid=" +
        this.user.username;
      //const UploadUrl= "https://k3c14giij6.execute-api.us-west-2.amazonaws.com/dev/fileUpload?type="+documentCode;
      const dbUrl = "https://api.upxtopcoder.com/V2/add";
      // const dbUrl =
      //   "https://7ba1ctdnue.execute-api.us-west-2.amazonaws.com/dev/add";
      try {
        const resp = await axios.post(UploadUrl, formData);
       // console.log(resp);
        this.confirmNum = resp.data.body.confirmNum;
        this.S3Url = resp.data.body.ImageURL;

        const itemData = {
          DocId: resp.data.body.confirmNum,
          PersonId: this.user.username,
          FirstName: this.fName,
          LastName: this.lName,
          email: this.email,
          age: this.age,
          title: this.title,
          DocType: this.doctype,
          DocCode: this.doccode,
          S3Url: resp.data.body.ImageURL,
          DocStatus: "Pending Review",
        };
        //console.log("ITem to be uploaded:", itemData);
         await axios.post(dbUrl, itemData);
        //console.log(resp2);
        this.message = resp;
        this.file = "";
        this.error = false;
        this.fexists = false;
        this.UploadSuccess = true;
      } catch (err) {
        this.message = err;
        this.error = true;
      }
    },
  },
};
</script>

<style scoped>
.fileUpload-form {
  width: 700px;
  margin: 30px auto;
  border: 1px solid #eee;
  padding: 20px;
  box-shadow: 0 2px 3px #ccc;
  align-content: left;
}

.input {
  margin: 10px auto;
}

.input label {
  display: block;
  color: #4e4e4e;
  margin-bottom: 6px;
}

.input.inline label {
  display: inline;
}

.input input {
  font: inherit;
  width: 100%;
  padding: 6px 12px;
  box-sizing: border-box;
  border: 1px solid #ccc;
}

.input.inline input {
  width: auto;
}

.input input:focus {
  outline: none;
  border: 1px solid #521751;
  background-color: #eee;
}

.input select {
  border: 1px solid #ccc;
  font: inherit;
}

.comments button {
  border: 1px solid #521751;
  background: #521751;
  color: white;
  padding: 6px;
  font: inherit;
  cursor: pointer;
}

.comments button:hover,
.comments button:active {
  background-color: #8d4288;
}

.comments input {
  width: 90%;
}

.submit button {
  border: 1px solid #521751;
  color: #521751;
  padding: 10px 20px;
  font: inherit;
  cursor: pointer;
  margin-top:10px;
}

.submit button:hover,
.submit button:active {
  background-color: #521751;
  color: white;
}

.submit button[disabled],
.submit button[disabled]:hover,
.submit button[disabled]:active {
  border: 1px solid #ccc;
  background-color: transparent;
  color: #ccc;
  cursor: not-allowed;
}
.dropzone {
  min-height: 100px;
  padding: 10px 10px;
  position: relative;
  cursor: pointer;
  outline: 2px dashed grey;
  outline-offset: -10px;
  background:lightblue;
}
.input-file {
  opacity: 0;
  width: 100%;
  height: 100px;
  position: absolute;
  cursor: pointer;
}
.dropzone .call-to-action {
  font-size: 2rem;
  text-align: center;
  padding: 25px;
}
.message {
  margin: 10px;
}
</style>
