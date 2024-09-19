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
  <!--
    <p>{{ calculateFirstSizeInMB }}</p>
    <p>{{ calculateSecondSizeInMB }}</p>  
  -->
  <button type="button" @click="compareTheFiles()">Compare the files</button>

  <div>
    <label for="oldList">Search in old List</label>
    <input
      type="text"
      id="oldList"
      placeholder="Type in the ID"
      v-model.trim="searchedIdOld"
    />
    <button type="button" @click="searchInNewDatabase('old')">Search</button>
    <p v-text="returnedOldString"></p>
  </div>
  <div>
    <label for="newList">Search in new List</label>
    <input
      type="text"
      id="newList"
      placeholder="Type in the ID"
      v-model.trim="searchedIdNew"
    />
    <button type="button" @click="searchInNewDatabase('new')">Search</button>
    <p v-text="returnedNewString"></p>
  </div>

  <h2>First File</h2>

  <table v-if="Object.keys(firstFinalBase).length !== 0">
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
  <table v-if="Object.keys(secondFinalBase).length !== 0">
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
import Papa from "papaparse";

export default {
  data() {
    return {
      creationDateOfFirstZipFile: 0,
      creationDateOfSecondZipFile: 0,
      firstFileName: "",
      firstInputElement: null,
      secondFileName: "",
      secondInputElement: null,
      firstNewDatabase: "",
      firstFinalBase: {},
      secondNewDatabase: "",
      secondFinalBase: {},
      deleteVinyls: [],
      sameKeys: [],
      newVinyls: [],
      changedVinyls: {},
      searchedIdOld: "",
      returnedOldString:
        "If you typed in the ID and clicked on the Search-button and still nothing is displayed, then you typed in a non-existant/wrong ID",
      searchedIdNew: "",
      returnedNewString:
        "If you typed in the ID and clicked on the Search-button and still nothing is displayed, then you typed in a non-existant/wrong ID",
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
    searchInNewDatabase(whichOne) {
      if (whichOne === "old") {
        console.log(this.searchedIdOld);
        const indexOfSearchedID = this.firstNewDatabase.indexOf(
          this.searchedIdOld
        );
        console.log(indexOfSearchedID);
        const indexOfEndOfVinylData = this.firstNewDatabase.indexOf(
          "\n",
          indexOfSearchedID
        );
        console.log(indexOfEndOfVinylData);
        const wantedData = this.firstNewDatabase.substring(
          indexOfSearchedID,
          indexOfEndOfVinylData
        );
        this.returnedOldString = wantedData;
      } else {
        console.log(this.searchedIdNew);
        const indexOfSearchedID = this.secondNewDatabase.indexOf(
          this.searchedIdNew
        );
        console.log(indexOfSearchedID);
        const indexOfEndOfVinylData = this.secondNewDatabase.indexOf(
          "\n",
          indexOfSearchedID
        );
        console.log(indexOfEndOfVinylData);
        const wantedData = this.secondNewDatabase.substring(
          indexOfSearchedID,
          indexOfEndOfVinylData
        );
        this.returnedNewString = wantedData;
      }
    },
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

                  // Papa Parse zum Verarbeiten des CSV-Inhalts
                  Papa.parse(content, {
                    header: true, // Wenn du Kopfzeilen in der CSV hast, kannst du diese Option setzen
                    skipEmptyLines: true, // Leere Zeilen überspringen
                    complete: (results) => {
                      const dataFromArrayIntoObject = {};
                      for (let entry of results.data) {
                        delete entry.accept_offer;
                        delete entry.external_id;
                        delete entry.format_quantity;
                        delete entry.location;
                        delete entry.quantity;
                        delete entry.weight;
                        dataFromArrayIntoObject["" + entry.listing_id] = entry;
                      }
                      // Hier wird das geparste CSV gespeichert
                      if (event.target.id === "firstInput") {
                        mainThis.firstNewDatabase = content;
                        console.log(results);
                        mainThis.firstFinalBase = dataFromArrayIntoObject;
                      } else {
                        mainThis.secondNewDatabase = content;
                        console.log(results);
                        mainThis.secondFinalBase = dataFromArrayIntoObject;
                      }
                    },
                    error: (error) => {
                      console.error("Error parsing CSV:", error);
                    },
                  });
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
    compareTheFiles() {
      for (let theKey in this.firstFinalBase) {
        if (this.secondFinalBase.hasOwnProperty(theKey)) {
          this.sameKeys.push(theKey);
        } else {
          this.deleteVinyls.push(theKey);
        }
      }
      for (let theKey in this.secondFinalBase) {
        if (!this.firstFinalBase.hasOwnProperty(theKey)) {
          this.newVinyls.push(theKey);
        }
      }
      for (let theKey of this.sameKeys) {
        const vinylInOldList = this.firstFinalBase["" + theKey];
        const vinylInNewList = this.secondFinalBase["" + theKey];
        let majorPriceDifference = false;
        if (vinylInOldList.price !== vinylInNewList.price) {
          const oldPrice = Number(vinylInOldList.price);
          const newPrice = Number(vinylInNewList.price);
          const priceDifference = Math.abs(oldPrice - newPrice);
          if (priceDifference >= 0.11) {
            majorPriceDifference = true;
          }
        }
        if (
          (vinylInOldList.media_condition !== vinylInNewList.media_condition ||
            vinylInOldList.sleeve_condition !==
              vinylInNewList.sleeve_condition ||
            vinylInOldList.comments !== vinylInNewList.comments ||
            majorPriceDifference === true) &&
          vinylInOldList.status === vinylInNewList.status
        ) {
          this.changedVinyls["" + theKey] = vinylInNewList;
        } else if (
          vinylInOldList.status === "For Sale" &&
          vinylInNewList.status === "Sold"
        ) {
          this.deleteVinyls.push(theKey);
        } else if (
          vinylInOldList.status === "Sold" &&
          vinylInNewList.status === "For Sale"
        ) {
          this.deleteVinyls.push(theKey);
          this.newVinyls.push(theKey);
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

.thePartition {
  margin-top: 7rem;
}

.input-area {
  padding: 1rem;
  padding-bottom: 7rem;
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

input + button {
  margin-left: 1rem;
}
</style>
