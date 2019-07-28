<template>
  <div id="app">
    <h1>Vue Prism Editor</h1>
    <button @click="save">Save</button>
    <h2>id: {{id}}</h2>
    <h2>
      title:
      <input type="text" v-model="title" />
    </h2>
    <input type="checkbox" v-model="lineNumbers" /> Linenumbers
    <div class="editor-container">
      <prism-editor v-model="code" language="js" :line-numbers="lineNumbers" class="my-editor" />
    </div>

    <h1>Autosize</h1>
    <p>
      don't use
      <b>min-height</b> or
      <b>height</b> you could still define
      <b>max-height</b>
    </p>
    <prism-editor v-model="code" language="js" />
  </div>
</template>

<script>
import PrismEditor from "vue-prism-editor";
import axios from "axios";
const root = "https://api.myjson.com/bins";
const postData = async data => {
  let response = await axios.post(root, data);
  let id = response.data.uri.split("/bins/")[1];
  return id;
};
const getData = async id => {
  let response = await axios.get(root + "/" + id);
  return response.data;
};
const putData = async (data, id) => {
  let response = await axios.put(root + "/" + id, data);
  return response.data;
};
export default {
  name: "App",
  data() {
    return {
      id: "",
      title: "",
      code: null,
      lineNumbers: true,
    };
  },
  components: {
    PrismEditor
  },
  watch: {
    title() {
      if(this.title.length > 0) document.title = this.title + ' - Vue-Note - ';
    }
  },
  methods: {
    save() {
      if (this.id.length > 0) {
        putData({ code: this.code, title: this.title }, this.id);
      } else {
        postData({ code: this.code, title: this.title }).then(result => {
          document.location.search = "?" + result;
        });
      }
    }
  },
  created() {
    let id = document.location.search;
    if (id.length > 0) {
      id = id.substr(1);
      this.id = id;
      getData(this.id).then(data => {
        this.code = data.code
        this.title = data.title || id
      });
    }
  }
};
</script>

<style>
.editor-container {
  margin: 0;
  padding: 0;
  height: 600px;
  font-size: 12px;
}
.editor {
  /* height: 100%; */
  font-size: 12px;
}
</style>
