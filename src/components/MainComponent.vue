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
    <button type="button" @click="searchInOld('old')">Search</button>
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
    <button type="button" @click="searchInOld('new')">Search</button>
    <p v-text="returnedNewString"></p>
  </div>

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
      oldWeirdVinylsTypeA: {},
      newWeirdVinylsTypeA: {},
      oldWeirdVinylsTypeB: {},
      newWeirdVinylsTypeB: {},
      creationDateOfFirstZipFile: 0,
      creationDateOfSecondZipFile: 0,
      firstFileName: "",
      firstInputElement: null,
      secondFileName: "",
      secondInputElement: null,
      firstNewDatabase: null,
      firstFinalBase: {},
      secondNewDatabase: null,
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
    searchInOld(whichOne) {
      if (whichOne === "old") {
        if (this.searchedIdOld.length < 9) {
          alert("The ID has to have at least 9 figures!!!");
        } else {
          for (let entry in this.firstNewDatabase) {
            if (this.searchedIdOld === entry) {
              this.returnedOldString =
                entry + ", " + this.firstNewDatabase[entry];
            }
          }
        }
      } else {
        if (this.searchedIdNew.length < 9) {
          alert("The ID has to have at least 9 figures!!!");
        } else {
          for (let entry in this.secondNewDatabase) {
            if (this.searchedIdNew === entry) {
              this.returnedNewString =
                entry + ", " + this.secondNewDatabase[entry];
            }
          }
        }
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
                  const regex = /(\d{9,}),/g;

                  const helpingArray = content.split(regex);
                  const objectForNewDatabase = {
                    headline: helpingArray[0],
                  };
                  for (let i = 1; i < helpingArray.length; i += 2) {
                    objectForNewDatabase["" + helpingArray[i]] =
                      helpingArray[i + 1];
                  }

                  let whichNewDataBase;
                  if (event.target.id === "firstInput") {
                    mainThis.firstNewDatabase = objectForNewDatabase;
                    whichNewDataBase = mainThis.firstNewDatabase;
                  } else {
                    mainThis.secondNewDatabase = objectForNewDatabase;
                    whichNewDataBase = mainThis.secondNewDatabase;
                  }

                  for (let platte in whichNewDataBase) {
                    // Split-Kommas, die nicht innerhalb von Anführungszeichen liegen
                    const result = whichNewDataBase[platte].match(
                      /(".*?"|[^",]+)(?=\s*,|\s*$)/g
                    );

                    if (result) {
                      // Entferne die umschließenden Anführungszeichen aus den Ergebnissen
                      const cleanedResult = result.map((item) =>
                        item.trim().replace(/^"(.*)"$/, "$1")
                      );

                      const shortenedCleanedResult = cleanedResult.slice(0, 12);

                      const newVinylObject = {
                        listing_id: platte,
                        artist: shortenedCleanedResult[0],
                        title: shortenedCleanedResult[1],
                        label: shortenedCleanedResult[2],
                        catno: shortenedCleanedResult[3],
                        format: shortenedCleanedResult[4],
                        release_id: shortenedCleanedResult[5],
                        status: shortenedCleanedResult[6],
                        price: shortenedCleanedResult[7],
                        listed: shortenedCleanedResult[8],
                        comments: shortenedCleanedResult[9],
                        media_condition: shortenedCleanedResult[10],
                        sleeve_condition: shortenedCleanedResult[11],
                      };

                      if (newVinylObject.status === undefined) {
                        newVinylObject.originalString =
                          whichNewDataBase[platte];
                        if (event.target.id === "firstInput") {
                          mainThis.oldWeirdVinylsTypeA["" + platte] =
                            newVinylObject;
                        } else {
                          mainThis.newWeirdVinylsTypeA["" + platte] =
                            newVinylObject;
                        }
                      } else if (newVinylObject.status.length > 15) {
                        newVinylObject.originalString =
                          whichNewDataBase[platte];
                        if (event.target.id === "firstInput") {
                          mainThis.oldWeirdVinylsTypeB["" + platte] =
                            newVinylObject;
                        } else {
                          mainThis.newWeirdVinylsTypeB["" + platte] =
                            newVinylObject;
                        }
                      } else {
                        if (event.target.id === "firstInput") {
                          mainThis.firstFinalBase["" + platte] = newVinylObject;
                        } else {
                          mainThis.secondFinalBase["" + platte] =
                            newVinylObject;
                        }
                      }
                    } else {
                      console.error("No match found for:", platte);
                    }
                  }
                  const howManyOldWeirdVinylsTypeA = Object.keys(
                    mainThis.oldWeirdVinylsTypeA
                  ).length;
                  const howManyNewWeirdVinylsTypeA = Object.keys(
                    mainThis.newWeirdVinylsTypeA
                  ).length;
                  console.log(
                    howManyOldWeirdVinylsTypeA + " old weird Vinyls Type A"
                  );
                  console.log(
                    howManyNewWeirdVinylsTypeA + " new weird Vinyls Type A"
                  );
                  const howManyOldWeirdVinylsTypeB = Object.keys(
                    mainThis.oldWeirdVinylsTypeB
                  ).length;
                  const howManyNewWeirdVinylsTypeB = Object.keys(
                    mainThis.newWeirdVinylsTypeB
                  ).length;
                  console.log(
                    howManyOldWeirdVinylsTypeB + " old weird Vinyls Type B"
                  );
                  console.log(
                    howManyNewWeirdVinylsTypeB + " new weird Vinyls Type B"
                  );
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
      console.log(this.deleteVinyls.length);
      console.log(this.sameKeys.length);
      // console.log(this.sameKeys);
      for (let theKey in this.secondFinalBase) {
        if (!this.firstFinalBase.hasOwnProperty(theKey)) {
          this.newVinyls.push(theKey);
        }
      }
      console.log(this.newVinyls.length);
      for (let theKey of this.sameKeys) {
        const vinylInOldList = Object.values(this.firstFinalBase["" + theKey]);
        const vinylInNewList = Object.values(this.secondFinalBase["" + theKey]);
        for (let i = 0; i < 13; i++) {
          if (vinylInOldList[i] !== vinylInNewList[i]) {
            let majorPriceDifference = false;
            if (i === 7) {
              console.log(theKey);
              console.log(vinylInOldList);
              console.log(vinylInNewList);
              console.log(vinylInOldList[i]);
              console.log(vinylInNewList[i]);
            }
            if (i === 8) {
              // console.log("Preis");
              const oldPriceInNumber = Number(vinylInOldList[8]);
              const newPriceInNumber = Number(vinylInNewList[8]);
              // console.log(oldPriceInNumber);
              // console.log(newPriceInNumber);
              const priceDiffenrence = Math.abs(
                oldPriceInNumber - newPriceInNumber
              );
              // console.log(priceDiffenrence);
              if (priceDiffenrence >= 0.11) {
                majorPriceDifference = true;
              }
              // 899534124
            }
            // if (i === 10) {
            //   console.log("Comments");
            // }
            // if (i === 11) {
            //   console.log("media_condition");
            // }
            // if (i === 12) {
            //   console.log("sleeve_condition");
            // }
            if (
              (i === 8 && majorPriceDifference === true) ||
              i === 10 ||
              i === 11 ||
              i === 12
            ) {
              // console.log(theKey);
              // console.log(this.secondFinalBase["" + theKey]);
              // console.log(vinylInNewList);
              // console.log(vinylInOldList[i]);
              // console.log(vinylInNewList[i]);
              // const newObject = {
              //   listing_id:
              // }
              this.changedVinyls["" + theKey] =
                this.secondFinalBase["" + theKey];
            }
            // console.log("---------------------------------------");
          }
        }
      }
      console.log(Object.keys(this.changedVinyls).length);
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
