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
    };
  },
  methods: {
    loadTheFile(event) {
      const file = event.target.files[0];
      console.log(file);

      if (file) {
        console.log(file.size);
        let theSize = file.size;
        if (theSize < 1024) {
          theSize = `${theSize} Bytes`;
        } else if (theSize < 1024 * 1024) {
          theSize = `${(theSize / 1024).toFixed(2)} KB`;
        } else {
          theSize = `${(theSize / (1024 * 1024)).toFixed(2)} MB`;
        }
        console.log(theSize);

        const reader = new FileReader();
        reader.onload = function (e) {
          const arrayBuffer = e.target.result;
          const jszip = new JSZip();
          jszip
            .loadAsync(arrayBuffer)
            .then((zip) => {
              zip.forEach((relativePath, file) => {
                file.async("string").then((content) => {
                  // Ausgabe des Dateinamens und Inhalts
                  // console.log("Datei:", relativePath);
                  // console.log("Inhalt:", content);
                  // console.log(typeof content);
                  const theParagraph = document.querySelector("p");
                  theParagraph.innerText = "content";
                  const blob = new Blob([content], { type: "text/plain" });
                  const sizeInBytes = blob.size;
                  console.log(sizeInBytes);
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
