<template>
  <h1>{{ msg }}</h1>
  <div class="input-area">
    <div class="first-input-area">
      <label for="firstInput">Lade die <span>erste</span> CSV-Datei</label>
      <input
        id="firstInput"
        type="file"
        accept=".zip"
        @change="loadTheFile($event)"
      />
    </div>
    <div class="second-input-area">
      <label for="secondInput">Lade die <span>zweite</span> CSV-Datei</label>
      <input
        id="secondInput"
        type="file"
        accept=".zip"
        @change="loadTheFile($event)"
      />
    </div>
  </div>
  <p>{{ calculateFirstSizeInMB }}</p>
  <p>{{ calculateSecondSizeInMB }}</p>
  <button type="button" @click="clearFile('first')">Delete File 1</button>
  <button type="button" @click="clearFile('second')">Delete File 2</button>
  <h2>First File</h2>
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
  <h2>Second File</h2>
  <table v-if="secondFinalBase.length > 10">
    <tr v-for="(vinyl, index) of secondFinalBase" :key="index">
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
      firstFileName: "",
      firstInputElement: null,
      secondFileName: "",
      secondInputElement: null,
      firstNewDatabase: null,
      firstFinalBase: [],
      secondNewDatabase: null,
      secondFinalBase: [],
    };
  },
  computed: {
    calculateFirstSizeInMB() {
      // Das JSON-Objekt in einen String umwandeln
      const jsonString = JSON.stringify(this.firstFinalBase);
      // Die Länge des Strings berechnen
      const byteSize = new Blob([jsonString]).size;
      // Die Größe in Megabytes umwandeln
      const sizeInMB = byteSize / (1024 * 1024);
      return "The first CSV-File is " + sizeInMB + " MB big!";
    },
    calculateSecondSizeInMB() {
      // Das JSON-Objekt in einen String umwandeln
      const jsonString = JSON.stringify(this.secondFinalBase);
      // Die Länge des Strings berechnen
      const byteSize = new Blob([jsonString]).size;
      // Die Größe in Megabytes umwandeln
      const sizeInMB = byteSize / (1024 * 1024);
      return "The first CSV-File is " + sizeInMB + " MB big!";
    },
  },
  methods: {
    loadTheFile(event) {
      const mainThis = this;
      console.log(event.target.files[0].name);

      if (
        event.target.id === "firstInput" &&
        event.target.files[0].name === this.secondFileName
      ) {
        alert(
          "You choose this file already for the other input. Please select a different file!"
        );
        event.target.value = "";
        return;
      } else if (
        event.target.id === "secondInput" &&
        event.target.files[0].name === this.firstFileName
      ) {
        alert(
          "You choose this file already for the other input. Please select a different file!"
        );
        event.target.value = "";
        return;
      }

      const file = event.target.files[0];
      console.log(file);

      if (event.target.id === "firstInput") {
        mainThis.firstInputElement = event.target;
        mainThis.firstFileName = event.target.files[0].name;
      } else {
        mainThis.secondInputElement = event.target;
        mainThis.secondFileName = event.target.files[0].name;
      }

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
                  console.log(typeof content);
                  console.log(mainThis.theDatabases);
                  const regex = /(?=\d{9},)/g;

                  let whichNewDataBase;
                  if (event.target.id === "firstInput") {
                    mainThis.firstNewDatabase = content.split(regex);
                    whichNewDataBase = mainThis.firstNewDatabase;
                  } else {
                    mainThis.secondNewDatabase = content.split(regex);
                    whichNewDataBase = mainThis.secondNewDatabase;
                  }

                  for (let platte of whichNewDataBase) {
                    // Split-Kommas, die nicht innerhalb von Anführungszeichen liegen
                    const result = platte.match(/(".*?"|[^",]+)(?=\s*,|\s*$)/g);

                    if (result) {
                      // Entferne die umschließenden Anführungszeichen aus den Ergebnissen
                      const cleanedResult = result.map((item) =>
                        item.trim().replace(/^"(.*)"$/, "$1")
                      );

                      const shortenedCleanedResult = cleanedResult.slice(0, 13);

                      if (event.target.id === "firstInput") {
                        mainThis.firstFinalBase.push(shortenedCleanedResult);
                      } else {
                        mainThis.secondFinalBase.push(shortenedCleanedResult);
                      }
                    } else {
                      console.error("No match found for:", platte);
                    }
                  }
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
    clearFile(whichOne) {
      if (whichOne === "first") {
        (this.firstFileName = ""), (this.firstNewDatabase = null);
        this.firstFinalBase = [];

        if (this.firstInputElement) {
          this.firstInputElement.value = "";
        }
      } else {
        this.secondFileName = "";
        this.secondNewDatabase = null;
        this.secondFinalBase = [];

        if (this.secondInputElement) {
          this.secondInputElement.value = "";
        }
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

.input-area,
.first-input-area,
.second-input-area {
  margin-top: 0;
}

.second-input-area {
  margin-block: 2rem;
}

label {
  margin-right: 1rem;
  font-weight: 600;
  font-size: 1.5rem;
}

span {
  color: red;
  text-decoration: underline;
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
  margin-top: 2rem;
}
</style>
