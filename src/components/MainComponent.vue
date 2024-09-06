<template>
  <h1>{{ msg }}</h1>
  <input type="file" accept=".zip" @change="loadTheFile($event)" />
  <p></p>
</template>

<script>
import JSZip from "jszip";

export default {
  data() {
    return {
      csvContent: "",
      theDatabases: [],
    };
  },
  methods: {
    loadTheFile(event) {
      const file = event.target.files[0];
      console.log(file);
      const mainThis = this;

      if (file) {
        const reader = new FileReader();
        reader.onload = function (e) {
          const arrayBuffer = e.target.result;
          const jszip = new JSZip();
          jszip
            .loadAsync(arrayBuffer)
            .then((zip) => {
              zip.forEach((relativePath, file) => {
                file.async("string").then((content) => {
                  console.log("Datei:", relativePath);
                  // console.log("Inhalt:", content);
                  console.log(typeof content);
                  if (!window.indexedDB) {
                    alert("Sorry! Your browser does not support IndexedDB");
                  }
                  console.log(mainThis.theDatabases);
                  if (mainThis.theDatabases.length === 0) {
                    mainThis.theDatabases.push(1);
                  } else {
                    const previousLastNumberOfDatabases =
                      mainThis.theDatabases[mainThis.theDatabases.length - 1];
                    let newLastNumberOfDatabases =
                      previousLastNumberOfDatabases + 1;
                    mainThis.theDatabases.push(newLastNumberOfDatabases);
                  }
                  // TODO: Füge indexedDB hinzu
                });
              });
            })
            .catch((err) => {
              console.error("Fehler beim Entpacken der ZIP-Datei:", err);
            });
        };
        reader.readAsArrayBuffer(file);
      }
    },
  },
  props: {
    msg: String,
  },
};
</script>

<style scoped>
div {
  margin-top: 5rem;
}
</style>
