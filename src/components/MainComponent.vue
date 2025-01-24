<template>
  <div class="main-area">
    <div
      v-show="progressTextDisplay"
      class="progess-container"
      :style="{
        backgroundColor: progressTextBackgroundColor,
      }"
    >
      {{ progressText }}
      <button
        type="button"
        class="progess-close-button"
        :style="{ display: progressCloseDisplay }"
        @click="hideProgress()"
      >
        X
      </button>
    </div>
    <header>
      <h1>{{ msg }}</h1>
    </header>
    <main>
      <div class="input-area">
        <div class="first-input-area">
          <label for="firstInput">Lade die <span>ältere</span> Zip-Datei</label>
          <input
            id="firstInput"
            type="file"
            accept=".zip"
            @change="storeTheFile($event)"
          />
        </div>
        <button type="button" @click="clearFile('first')">Delete File 1</button>
        <div class="thePartition"></div>
        <div class="second-input-area">
          <label for="secondInput"
            >Lade die <span>neuere</span> Zip-Datei</label
          >
          <input
            id="secondInput"
            type="file"
            accept=".zip"
            @change="storeTheFile($event)"
          />
        </div>
        <button type="button" @click="clearFile('second')">
          Delete File 2
        </button>
      </div>
      <!--
      <p>{{ calculateFirstSizeInMB }}</p>
      <p>{{ calculateSecondSizeInMB }}</p>  
    -->
      <button type="button" @click="compareTheFiles()">
        Compare the files
      </button>

      <div>
        <label for="oldList">Search in old List</label>
        <input
          type="text"
          id="oldList"
          placeholder="Type in the ID"
          v-model.trim="searchedIdOld"
        />
        <button type="button" @click="searchInNewDatabase('old')">
          Search
        </button>
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
        <button type="button" @click="searchInNewDatabase('new')">
          Search
        </button>
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
    </main>
  </div>
</template>

<script>
import JSZip from "jszip";
import Papa from "papaparse";

export default {
  data() {
    return {
      progressText: "Please Wait 😉",
      progressTextDisplay: false,
      progressTextBackgroundColor: "lightseagreen",
      progressCloseDisplay: "none",
      creationDateOfFirstZipFile: 0,
      creationDateOfSecondZipFile: 0,
      firstFileName: "",
      firstInputElement: null,
      firstZipFile: null,
      secondZipFile: null,
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
    hideProgress() {
      if (this.progressText === "Done 🤩") {
        (this.progressTextDisplay = false),
          (this.progressTextBackgroundColor = "lightseagreen"),
          (this.progressText = "Please Wait 😉");
        this.progressCloseDisplay = "none";
      }
    },
    searchInNewDatabase(whichOne) {
      if (whichOne === "old") {
        const indexOfSearchedID = this.firstNewDatabase.indexOf(
          this.searchedIdOld
        );
        const indexOfEndOfVinylData = this.firstNewDatabase.indexOf(
          "\n",
          indexOfSearchedID
        );
        const wantedData = this.firstNewDatabase.substring(
          indexOfSearchedID,
          indexOfEndOfVinylData
        );
        this.returnedOldString = wantedData;
      } else {
        const indexOfSearchedID = this.secondNewDatabase.indexOf(
          this.searchedIdNew
        );
        const indexOfEndOfVinylData = this.secondNewDatabase.indexOf(
          "\n",
          indexOfSearchedID
        );
        const wantedData = this.secondNewDatabase.substring(
          indexOfSearchedID,
          indexOfEndOfVinylData
        );
        this.returnedNewString = wantedData;
      }
    },
    unpackAndParseTheFile(whichFile) {
      const mainThis = this;
      let file;
      if (whichFile === "oldFile") {
        file = this.firstZipFile;
      } else {
        file = this.secondZipFile;
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
                      if (whichFile === "oldFile") {
                        mainThis.firstNewDatabase = content;
                        mainThis.firstFinalBase = dataFromArrayIntoObject;
                      } else {
                        mainThis.secondNewDatabase = content;
                        mainThis.secondFinalBase = dataFromArrayIntoObject;
                      }
                      if (whichFile === "newFile") {
                        mainThis.compareTheFiles();
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
    storeTheFile(event) {
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

      if (event.target.id === "firstInput") {
        this.creationDateOfFirstZipFile = event.target.files[0].lastModified;
        if (
          this.creationDateOfSecondZipFile === 0 ||
          this.creationDateOfFirstZipFile < this.creationDateOfSecondZipFile
        ) {
          this.firstInputElement = event.target;
          this.firstFileName = event.target.files[0].name;
          this.firstZipFile = event.target.files[0];
        } else if (
          this.creationDateOfFirstZipFile > this.creationDateOfSecondZipFile
        ) {
          alert(
            "The upper Zip-file has to be older than the lower one. You can find the creation date in the file-name after 'inventory-'. Please insert the upper file in the lower input-field and the lower file in the upper input-field."
          );
          this.creationDateOfFirstZipFile = 0;
          this.firstZipFile = null;
          event.target.value = "";
          return;
        }
      } else {
        this.creationDateOfSecondZipFile = event.target.files[0].lastModified;
        if (
          this.creationDateOfFirstZipFile === 0 ||
          this.creationDateOfFirstZipFile < this.creationDateOfSecondZipFile
        ) {
          this.secondInputElement = event.target;
          this.secondFileName = event.target.files[0].name;
          this.secondZipFile = event.target.files[0];
        } else if (
          this.creationDateOfFirstZipFile > this.creationDateOfSecondZipFile
        ) {
          alert(
            "The upper Zip-file has to be older than the lower one. You can find the creation date in the file-name after 'inventory-'. Please insert the upper file in the lower input-field and the lower file in the upper input-field."
          );
          this.creationDateOfSecondZipFile = 0;
          this.secondZipFile = null;
          event.target.value = "";
          return;
        }
      }

      if (this.firstZipFile !== null && this.secondZipFile !== null) {
        this.progressTextDisplay = true;
        this.unpackAndParseTheFile("oldFile");
        this.unpackAndParseTheFile("newFile");
      }
    },
    clearFile(whichOne) {
      if (whichOne === "first") {
        this.firstFileName = "";
        this.firstNewDatabase = null;
        this.firstFinalBase = {};
        this.creationDateOfFirstZipFile = 0;
        this.firstZipFile = null;

        if (this.firstInputElement) {
          this.firstInputElement.value = "";
        }
      } else {
        this.secondFileName = "";
        this.secondNewDatabase = null;
        this.secondFinalBase = {};
        this.creationDateOfSecondZipFile = 0;
        this.secondZipFile = null;

        if (this.secondInputElement) {
          this.secondInputElement.value = "";
        }
      }
    },
    compareTheFiles() {
      this.deleteVinyls = [];
      this.newVinyls = [];
      this.sameKeys = [];
      this.changedVinyls = {};
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
      console.log(this.deleteVinyls.length + " old Vinyls/CDs were deleted");
      console.log(this.newVinyls.length + " new Vinyls/CDs were added");
      console.log(
        Object.keys(this.changedVinyls).length + " Vinyls/CDs were changed"
      );
      this.progressTextBackgroundColor = "lightsalmon";
      this.progressText = "Done 🤩";
      this.progressCloseDisplay = "block";
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

.progess-container {
  border: 0.25rem solid black;
  width: fit-content;
  font-size: 5rem;
  padding: 5rem;
  position: absolute;
  left: 30%;
  top: 10rem;
}

.progess-close-button {
  all: unset;
  font-size: 1rem;
  font-weight: 600;
  border: 0.15rem solid black;
  border-radius: 50%;
  width: 1.5rem;
  padding: 0.5rem;
  background-color: lightgray;
  position: absolute;
  top: 0.75rem;
  right: 0.75rem;
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
