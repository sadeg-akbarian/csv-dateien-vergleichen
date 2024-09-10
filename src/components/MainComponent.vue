<template>
  <h1>{{ msg }}</h1>
  <div class="input-area">
    <div class="first-input-area">
      <label for="firstInput">Lade die <span>ältere</span> Zip-Datei</label>
      <input
        id="firstInput"
        type="file"
        accept=".zip"
        @change="loadTheFile($event)"
      />
    </div>
    <button type="button" @click="clearFile('first')">Delete File 1</button>
    <div class="thePartition"></div>
    <div class="second-input-area">
      <label for="secondInput">Lade die <span>neuere</span> Zip-Datei</label>
      <input
        id="secondInput"
        type="file"
        accept=".zip"
        @change="loadTheFile($event)"
      />
    </div>
    <button type="button" @click="clearFile('second')">Delete File 2</button>
  </div>
  <p>{{ calculateFirstSizeInMB }}</p>
  <p>{{ calculateSecondSizeInMB }}</p>

  <h2>First File</h2>

  <table v-if="Object.keys(firstFinalBase).length > 10">
    <tr>
      <td>listing_id</td>
      <td>artist</td>
      <td>title</td>
      <td>label</td>
      <td>catno</td>
      <td>format</td>
      <td>release_id</td>
      <td>status</td>
      <td>price</td>
      <td>listed</td>
      <td>comments</td>
      <td>media_condition</td>
      <td>sleeve_condition</td>
    </tr>
    <tr v-for="(vinyl, id, index) in firstFinalBase" :key="id">
      <template v-if="index < 10">
        <td>{{ vinyl.listing_id }}</td>
        <td>{{ vinyl.artist }}</td>
        <td>{{ vinyl.title }}</td>
        <td>{{ vinyl.label }}</td>
        <td>{{ vinyl.catno }}</td>
        <td>{{ vinyl.format }}</td>
        <td>{{ vinyl.release_id }}</td>
        <td>{{ vinyl.status }}</td>
        <td>{{ vinyl.price }}</td>
        <td>{{ vinyl.listed }}</td>
        <td>{{ vinyl.comments }}</td>
        <td>{{ vinyl.media_condition }}</td>
        <td>{{ vinyl.sleeve_condition }}</td>
      </template>
    </tr>
  </table>
  <h2>Second File</h2>
  <table v-if="Object.keys(secondFinalBase).length > 10">
    <tr>
      <td>listing_id</td>
      <td>artist</td>
      <td>title</td>
      <td>label</td>
      <td>catno</td>
      <td>format</td>
      <td>release_id</td>
      <td>status</td>
      <td>price</td>
      <td>listed</td>
      <td>comments</td>
      <td>media_condition</td>
      <td>sleeve_condition</td>
    </tr>
    <tr v-for="(vinyl, id, index) in secondFinalBase" :key="id">
      <template v-if="index < 10">
        <td>{{ vinyl.listing_id }}</td>
        <td>{{ vinyl.artist }}</td>
        <td>{{ vinyl.title }}</td>
        <td>{{ vinyl.label }}</td>
        <td>{{ vinyl.catno }}</td>
        <td>{{ vinyl.format }}</td>
        <td>{{ vinyl.release_id }}</td>
        <td>{{ vinyl.status }}</td>
        <td>{{ vinyl.price }}</td>
        <td>{{ vinyl.listed }}</td>
        <td>{{ vinyl.comments }}</td>
        <td>{{ vinyl.media_condition }}</td>
        <td>{{ vinyl.sleeve_condition }}</td>
      </template>
    </tr>
  </table>
</template>

<script>
import JSZip from "jszip";

export default {
  data() {
    return {
      creationDateOfFirstZipFile: 0,
      creationDateOfSecondZipFile: 0,
      csvContent: "",
      theDatabases: [],
      firstFileName: "",
      firstInputElement: null,
      secondFileName: "",
      secondInputElement: null,
      firstNewDatabase: null,
      firstFinalBase: {},
      secondNewDatabase: null,
      secondFinalBase: {},
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
        mainThis.creationDateOfFirstZipFile =
          event.target.files[0].lastModified;
        if (
          mainThis.creationDateOfSecondZipFile === 0 ||
          mainThis.creationDateOfFirstZipFile <
            mainThis.creationDateOfSecondZipFile
        ) {
          mainThis.firstInputElement = event.target;
          mainThis.firstFileName = event.target.files[0].name;
        } else if (
          mainThis.creationDateOfFirstZipFile >
          mainThis.creationDateOfSecondZipFile
        ) {
          alert(
            "Die obere Zip-Datei muss älter sein als die untere! Das Datum findest du im Dateinamen nach 'inventory-'. Bitte füge die obere Datei in das untere Auswahlfeld und die untere Datei in das obere Auswahlfeld ein."
          );
          mainThis.creationDateOfFirstZipFile = 0;
          event.target.value = "";
          console.log("qqqqqqqqqqqqq");
          return;
        }
      } else {
        mainThis.creationDateOfSecondZipFile =
          event.target.files[0].lastModified;
        if (
          mainThis.creationDateOfFirstZipFile === 0 ||
          mainThis.creationDateOfFirstZipFile <
            mainThis.creationDateOfSecondZipFile
        ) {
          mainThis.secondInputElement = event.target;
          mainThis.secondFileName = event.target.files[0].name;
        } else if (
          mainThis.creationDateOfFirstZipFile >
          mainThis.creationDateOfSecondZipFile
        ) {
          alert(
            "The upper Zip-file has to be older than the lower one. You can find the creation date in the file-name after 'inventory-'. Please insert the upper file in the lower input-field and the lower file in the upper input-field."
          );
          mainThis.creationDateOfSecondZipFile = 0;
          event.target.value = "";
          console.log("qqqqqqqqqqqqq");
          return;
        }
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

                      const newKey = shortenedCleanedResult[0];

                      const newVinylObject = {
                        listing_id: newKey,
                        artist: shortenedCleanedResult[1],
                        title: shortenedCleanedResult[2],
                        label: shortenedCleanedResult[3],
                        catno: shortenedCleanedResult[4],
                        format: shortenedCleanedResult[5],
                        release_id: shortenedCleanedResult[6],
                        status: shortenedCleanedResult[7],
                        price: shortenedCleanedResult[8],
                        listed: shortenedCleanedResult[9],
                        comments: shortenedCleanedResult[10],
                        media_condition: shortenedCleanedResult[11],
                        sleeve_condition: shortenedCleanedResult[12],
                      };

                      if (event.target.id === "firstInput") {
                        mainThis.firstFinalBase["" + newKey] = newVinylObject;
                      } else {
                        mainThis.secondFinalBase["" + newKey] = newVinylObject;
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
        this.firstFileName = "";
        this.firstNewDatabase = null;
        this.firstFinalBase = {};
        this.creationDateOfFirstZipFile = 0;

        if (this.firstInputElement) {
          this.firstInputElement.value = "";
        }
      } else {
        this.secondFileName = "";
        this.secondNewDatabase = null;
        this.secondFinalBase = {};
        this.creationDateOfSecondZipFile = 0;

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

.input-area,
.thePartition {
  border: 0.75rem solid blue;
  margin: 0;
  margin-top: 2rem;
}

.input-area {
  padding: 1rem;
  margin-bottom: 2rem;
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
}

button:nth-child(2) {
  margin-top: 2rem;
}
/* button:nth-child(1) */
</style>
