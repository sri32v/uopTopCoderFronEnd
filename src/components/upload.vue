<template>
  <div id="fileUpload">
    <h1>The User Page</h1>
    <div class="fileUpload-form" align="left">
      <form @submit.prevent="saveData" enctype="multipart/form-data">
        <div
          v-if="!hide"
          :class="`${error ? 'alert alert-danger' : 'alert alert-success'}`"
        >
          <div
            class="`${error ? 'alert alert-danger' : 'alert alert-success'}`"
          >
            {{ message }}
          </div>
        </div>
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
          <label for="age">Age</label>
          <input type="number" id="age" v-model.number="age" />
        </div>
        <div class="input">
          <label for="doccode">Document Code</label>
          <select id="doccode" v-model="doccode">
            <option value="TRS">UOPUS_Transcript</option>
            <option value="DL">UOPUS_Driving</option>
          </select>
        </div>
        <div class="input">
          <label for="doctype">Document Type</label>
          <select id="doctype" v-model="doctype">
            <option value="PDF">PDF</option>
            <option value="JPEG">JPEG</option>
          </select>
        </div>
        <div class="comments">
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
        <div class="input inline">
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
          {{ file.name }} <br>Size:{{file.size/1000}}kb
        </div>
         <div
          v-if="message"
          :class="`message ${error ? 'alert alert-danger' : 'alert alert-success'}`"
        >
          <div
            class="`message ${error ? 'alert alert-danger' : 'alert alert-success'}`"
          >
          FileUploaded Successfully. <br> ConfirmationNumber: {{ message }}
          </div>
        </div>

        <div class="submit">
          <button type="submit">Submit</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
//import dropzone from './components/dropzone.vue'
//import _ from "lodash";
import axios from 'axios';
export default {
  name:"Upload",
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
      fName:"",
      lName:"",
      age:"",
      doccode:"",
      doctype:"",
      terms:false,
      hide:true,
      confirmNum:"",
      S3Url:""
    };
  },
  // components:{dropzone},
  methods: {
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
      const MAX_SIZE = 500000;
      console.log(file);
      const tooLarge = file.size > MAX_SIZE;
      if (allowedTypes.includes(file.type) && !tooLarge) {
        this.file = file;
        this.error = false;
        this.fexists = true;
        this.message = "";
      } else {
        this.error = true;
        this.message = tooLarge
          ? `Too large. Max size is ${MAX_SIZE / 1000}kb`
          : "Only images,Pdfs are allowed";
      }
    },
    validate(file) {
      const MAX_SIZE = 1000000;
      const allowedTypes = ["image/jpeg", "image/png", "image/gif"];
      
      if (file.size > MAX_SIZE) {
        return `Max size: ${MAX_SIZE / 1000}Kb`;
      }
      if (!allowedTypes.includes(file.type)) {
        return "Not an Image.";
      }
      return "";
    },
    async saveData() {
    const formData=new FormData();

const documentCode='DRd';
formData.append('file',this.file);
const UploadUrl= "https://k3c14giij6.execute-api.us-west-2.amazonaws.com/dev/fileUpload?type="+documentCode;
const dbUrl="https://7ba1ctdnue.execute-api.us-west-2.amazonaws.com/dev/add";
try {

   const resp = await axios.post(UploadUrl,formData);
      const itemData={
    DocId:  "3112200",
     PersonId: 324533333,
       title: 'changed title',
       content: 'changed content',
       DocType:'pdf',
       DocCode:'UOPUS_Transcript',
   response:resp    
  }
  console.log(itemData);
  const resp2 = await axios.post(dbUrl,itemData);
  //this.message=resp.data.body.confirmNum;
  console.log(resp2);
  this.message=resp;
  this.file="";
  this.error=false;
  this.fexists = false;
 // this.S3Url=resp.body.imageUrl;
  //saveDataDb();
  this.$router.push({path: '/profile'})
 
} catch (err) {
  this.message=err;
  this.error=true;
  console.log(this.message);
}
    }
  },
  async saveDataDb() {
    const itemData={
    DocId: "33234243234232",
     PersonId: 23333,
       title: 'changed title',
       content: 'changed content',
       DocType:'pdf',
       DocCode:'UOPUS_Transcript',
  }
  console.log(itemData);
  const dbUrl="https://7ba1ctdnue.execute-api.us-west-2.amazonaws.com/dev/add";
  const config = {
    headers:{'Access-Control-Allow-Origin': '*'}
  }
  let data2= await axios(dbUrl,itemData,config).post().promise();
  console.log(data2);
  }
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
  background: lightgoldenrodyellow;
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
