<script setup>
import { ref, onMounted, watch } from "vue";
import axios from "axios";

//state for holding ref of texts,images and selectedFile
const images = ref([]);
const texts = ref([]);
const selectedFile = ref("");

// get all the images from the backend
const getImages = async () => {
  try {
    const data = await axios.get("http://localhost:8000/images");

    let id = 0;
    images.value = data.data.map((item) => {
      return {
        id: ++id,
        item: item,
      };
    });
    id = 0;
  } catch (error) {
    console.error(error);
  }
  console.log(images.value);
};

//tracks filechange on select image
function onfileChange(e) {
  selectedFile.value = e.target.files[0];
}

//upload it to the backend
function upload() {
  const formData = new FormData();
  if (selectedFile.value === "") return;
  formData.append("upload", selectedFile.value);
  // console.log(formData);
  axios
    .post("http://localhost:8000/uploads", formData)
    .then((res) => {
      console.log(res);
    })
    .catch((err) => {
      console.log(err);
    });
  //get the latest set of images
  getImages();
}

//get the text values and add it to the canvas
function addText(e) {
  const input = document.getElementById("addTextInput");
  if (input.value === "") return;
  texts.value = [...texts.value, input.value];
  localStorage.setItem("texts", JSON.stringify(texts.value));
  input.value = "";
}

//  hook to watch for  new texts and set it to localstorage
watch(texts, (newText) => {
  localStorage.setItem("texts", JSON.stringify(newText));
});

//lifecycle hook when our app renders first time
onMounted(() => {
  //getting images from the localstorage and set it to texts ref
  texts.value = JSON.parse(localStorage.getItem("texts")) || [];

  //get images from the backend to get initial render
  getImages();
  // images.value = JSON.parse(localStorage.getItem("images")) || [];
});

//this function allows drop on canvas
function allowDrop(ev) {
  ev.preventDefault();
}

//grabing image data
function drag(ev) {
  ev.dataTransfer.setData("text", ev.target.id);
}

//drop on target
function drop(ev) {
  ev.preventDefault();
  var data = ev.dataTransfer.getData("text");
  var element = document.getElementById(data);
  element.style.width = "100px";
  element.style.height = "100px";
  ev.target.appendChild(element);
}
</script>

<template>
  <head>
    <title>Mini Piktochart</title>
    <link
      rel="stylesheet"
      type="text/css"
      href="/node_modules/bootstrap/dist/css/bootstrap.min.css"
    />
    <link rel="stylesheet" type="text/css" href="/src/main.css" />
  </head>

  <body>
    <div class="h-100 w-100">
      <div class="row h-100 w-100 m-0">
        <div class="sidepane h-100 bg-dark text-white col-2">
          <div class="form mt-5">
            <h3>Form</h3>
            <hr />
            <div class="input-group">
              <div class="custom-file">
                <input
                  id="upload-image-input"
                  class="custom-file-input"
                  name="upload"
                  type="file"
                  @change="onfileChange"
                />
                <label class="custom-file-label" for="upload-image-input"
                  >Choose Image</label
                >
              </div>
              <div class="input-group-append">
                <button
                  @click="upload"
                  class="btn btn-info"
                  type="button"
                  id="upload-image"
                >
                  Upload
                </button>
              </div>
            </div>
            <!-- Upload Form here -->
          </div>
          <div class="assets w-100 mt-5">
            <h3>Assets</h3>
            <hr />
            <div class="text">
              <h4>Text</h4>
              <input id="addTextInput" type="text" class="form-control" />
              <button
                id="addText"
                @click="addText"
                class="btn btn-info btn-block mt-2"
              >
                Add Text
              </button>
            </div>
            <div class="image">
              <h4>Images</h4>
              <ul class="list-unstyled">
                <!-- List of images here -->
                <li v-for="image of images">
                  <img
                    :src="image.item"
                    @dragstart="drag($event)"
                    :id="image.id"
                    alt="item"
                  />
                </li>
              </ul>
            </div>
          </div>
        </div>
        <div class="canvas col-10">
          <div
            @drop="drop($event)"
            @dragover="allowDrop($event)"
            class="block mx-auto mt-3"
          >
            <div
              class="btn btn-info mb-4"
              style="width: 15rem; font-style: italic"
            >
              Texts Goes Here
            </div>
            <ul>
              <li style="list-style: none" v-for="text of texts">
                <div
                  :draggable="true"
                  class="btn btn-secondary mb-2 font-italic"
                  style="width: 10rem"
                >
                  {{ text }}
                </div>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </body>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
}

html,
body {
  width: 100%;
  height: 100%;
}

.editor {
  height: 100vh;
  width: 100vw;
}

#assetText {
  font-size: 24px;
}

.assets .text,
.assets .image {
  margin: 10px 0;
}

.assets .image ul li {
  width: 50px;
  height: 50px;
  margin-right: 5px;
  float: left;
  overflow: hidden;
}

.assets .image ul li img {
  width: 100%;
  height: 100%;
}

.canvas .block {
  position: relative;
  width: 800px;
  height: 800px;
  border: 1px solid;
  box-shadow: 0px 0px 30px -15px var(--gray-dark);
}

.item {
  border: 1px solid transparent;
  position: absolute;
}

.item.selected {
  border-color: var(--cyan);
}
</style>
