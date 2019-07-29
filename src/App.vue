<template>
  <div id="app">
    <div class="header">
      <h4>
        <a href="/vue-note/">New</a>
        <span class="margin">
          title:
          <input class="text" type="text" v-model="title" />
        </span> |
        <span class="margin">id: {{id}}</span>
        <button class="margin" @click="save">{{btnLabel}}</button>
      </h4>
    </div>

    <div class="editor-container">
      <input type="checkbox" v-model="lineNumbers" /> Linenumbers
      <prism-editor v-model="code" language="js" :line-numbers="lineNumbers" class="my-editor" />
    </div>
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
  return id;
};
export default {
  name: "App",
  components: {
    PrismEditor
  },
  data() {
    return {
      btnLabel: "Save",
      id: "",
      title: "",
      code: null,
      lineNumbers: true
    };
  },

  watch: {
    title() {
      if (this.title.length > 0) document.title = this.title + " - Vue-Note";
    }
  },
  methods: {
    save() {
      if (this.id.length > 0) {
        putData({ code: this.code, title: this.title }, this.id).then(
          result => {
            document.location.search = "?" + result;
          }
        );
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
        this.code = data.code;
        this.title = data.title || id;
        this.btnLabel = "Update";
      });
    } else {
      this.title = "New Note";
    }
  }
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}

#app {
  display: inline-grid;
  grid-template-rows: 3em calc(100vh - 3em);
  background-color: #666;
  overflow: hidden;
  height: 100vh;
  width: 100vw;
  margin: 0;
  padding: 0;
}
.header {
  padding: 0px 10px;
}
button {
  padding: 2px 15px;
  border-radius: 10px;
  border-color: rgba(10, 148, 67, 0.644);
  background-color: rgb(25, 78, 47);
  color: #aaa;
  font-weight: 500;
}
button:focus {
  outline: 0;
}
input.text {
  width: 8em;
}
.editor-container {
  overflow: hidden;
  flex: auto;
  margin: 5px;
  padding: 5px;
  padding-bottom: 2em;
  font-size: 13px;
}
.editor {
  overflow-x: scroll;
}
.margin {
  margin: 0px 5px;
}
</style>
