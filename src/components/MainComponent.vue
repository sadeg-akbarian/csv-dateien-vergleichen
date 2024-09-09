<template>
  <h1>{{ msg }}</h1>
  <input type="file" accept=".zip" @change="loadTheFile($event)" />
  <p>{{ calculateSizeInMB }}</p>
  <button type="button" @click="clearFile">Delete File 1</button>
  <button type="button">Delete File 2</button>
  <table v-if="firstFinalBase.length > 10">
    <tr v-for="(vinyl, index) of firstFinalBase" :key="index">
      <template v-if="index < 10">
        <td>{{ vinyl[0] }}</td>
        <td>{{ vinyl[1] }}</td>
        <td>{{ vinyl[2] }}</td>
        <td>{{ vinyl[3] }}</td>
        <td>{{ vinyl[4] }}</td>
        <td>{{ vinyl[5] }}</td>
        <td>{{ vinyl[6] }}</td>
        <td>{{ vinyl[7] }}</td>
        <td>{{ vinyl[8] }}</td>
        <td>{{ vinyl[9] }}</td>
        <td>{{ vinyl[11] }}</td>
        <td>{{ vinyl[12] }}</td>
      </template>
    </tr>
  </table>
</template>

<script>
import JSZip from "jszip";

export default {
  data() {
    return {
      csvContent: "",
      theDatabases: [],
      fileInputElement: null,
      firstNewDatabase: null,
      firstFinalBase: [],
    };
  },
  computed: {
    calculateSizeInMB() {
      // Das JSON-Objekt in einen String umwandeln
      const jsonString = JSON.stringify(this.firstFinalBase);
      // Die Länge des Strings berechnen
      const byteSize = new Blob([jsonString]).size;
      // Die Größe in Megabytes umwandeln
      const sizeInMB = byteSize / (1024 * 1024);
      return "It is " + sizeInMB + " MB big!";
    },
  },
  methods: {
    loadTheFile(event) {
      const file = event.target.files[0];
      console.log(file);
      const mainThis = this;
      mainThis.fileInputElement = event.target;

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
                  const regex = /(?=\d{9},)/g;
                  mainThis.firstNewDatabase = content.split(regex);
                  // console.log();

                  for (let platte of mainThis.firstNewDatabase) {
                    // Split-Kommas, die nicht innerhalb von Anführungszeichen liegen
                    const result = platte.match(/(".*?"|[^",]+)(?=\s*,|\s*$)/g);

                    if (result) {
                      // Entferne die umschließenden Anführungszeichen aus den Ergebnissen
                      const cleanedResult = result.map((item) =>
                        item.trim().replace(/^"(.*)"$/, "$1")
                      );

                      const shortenedCleanedResult = cleanedResult.slice(0, 13);
                      mainThis.firstFinalBase.push(shortenedCleanedResult);
                    } else {
                      console.error("No match found for:", platte);
                    }
                  }

                  // TODO: Im mainThis.firstFinalBase müssen die items in key-value-Paare umgewandelt werden, bei denen der key die ID ist
                  // TODO: und der value ein object, der die restlichen Daten der Platte enthält

                  // :::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

                  // const request = indexedDB.open("CSV-Databases", 1);
                  // request.onupgradeneeded = (event) => {
                  //   mainThis.firstNewDatabase = event.target.result;
                  //   if (
                  //     !mainThis.firstNewDatabase.objectStoreNames.contains("myStore")
                  //   ) {
                  //     mainThis.firstNewDatabase.createObjectStore("myStore", {
                  //       keyPath: "id",
                  //       autoIncrement: true,
                  //     });
                  //   }
                  // };
                  // request.onsuccess = (event) => {
                  //   mainThis.firstNewDatabase = event.target.result;
                  //   console.log("Database opened successfully");
                  //   console.log(event);
                  //   console.log(event.target);

                  //   // :::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

                  //   const transaction = mainThis.firstNewDatabase.transaction(
                  //     ["myStore"],
                  //     "readwrite"
                  //   );
                  //   const store = transaction.objectStore("myStore");
                  //   store.add({ content });

                  //   transaction.oncomplete = () => {
                  //     console.log("Data added");
                  //     const transaction = mainThis.firstNewDatabase.transaction(
                  //       ["myStore"],
                  //       "readonly"
                  //     );
                  //     const store = transaction.objectStore("myStore");
                  //     const request = store.get(1);

                  //     request.onsuccess = (event) => {
                  //       // console.log(event.target.result.content);
                  //       const regex = /(?=\d{9},)/g;
                  //       mainThis.firstFinalBase =
                  //         event.target.result.content.split(regex);
                  //     };
                  //   };

                  //   // :::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
                  // };

                  // request.onerror = (event) => {
                  //   console.error("Database error:", event.target.errorCode);
                  // };
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
    clearFile() {
      this.firstNewDatabase = null;
      this.firstFinalBase = [];

      if (this.fileInputElement) {
        this.fileInputElement.value = "";
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

table {
  margin-top: 3rem;
  border-collapse: collapse;
}

table,
td {
  border: 2px solid black;
}

button {
  border: 1px solid black;
  margin-right: 1rem;
}
</style>
