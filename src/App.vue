<template>
  <h1>SQL Insert</h1>

  <div class="select-insert-option">
    <button class="sql-button" @click="toggleTWebsites">
      T-Websites & Content
    </button>
    <button class="sql-button" @click="toggleTFiles">
      T-Files
    </button>
  </div>
  <div class="page-wrapper">
    <div class="webcontent-selection">
      <div>
        <label>Table:</label>
        <div class="dropdown">
          <div class="dropdown-selected" @click="toggleTableDropdown">{{ selectedTableWeb.name }}</div>
          <div class="dropdown-options" v-if="showTableDropdown">
            <div v-for="option in tableOptions" :key="option.name" @click="selectTable(option)">
              {{ option.name }}
            </div>
          </div>
        </div>

      </div>

      <div>
        <label for="websiteId">Website ID:</label>
        <input id="websiteId" type="number" v-model="websiteId">
      </div>
      <div>
        <label for="link">Link:</label>
        <input id="link" type="text" v-model="link">
      </div>
      <div>
        <input type="checkbox" id="hasVueSite" v-model="hasVueSite">
        <label for="hasVueSite">Has Vue Site</label>
      </div>
      <button @click="generateInsertStatements">Generate SQL</button>
    </div>
  </div>


  <div v-if="showTFiles" class="page-wrapper">
    <div class="settings">
      <div>
        <label>Website:</label>
        <div class="dropdown">
          <div class="dropdown-selected" @click="toggleFolderDropdown">{{ selectedFolder }}</div>
          <div class="dropdown-options" v-if="showFolderDropdown">
            <div v-for="table in tables" :key="table.folder" @click="selectFolder(table.folder)">{{
                table.folder
              }}
            </div>
          </div>
        </div>
      </div>

      <div>
        <label>Table:</label>
        <div class="dropdown">
          <div class="dropdown-selected">{{ selectedTable }}</div>
        </div>
      </div>

      <div>
        <label>Language ID:</label>
        <div class="dropdown">
          <div class="dropdown-selected" @click="toggleLanguageDropdown">{{ selectedLanguageId }}</div>
          <div class="dropdown-options" v-if="showLanguageDropdown">
            <div v-for="id in languageIds" :key="id" @click="selectLanguage(id)">{{ id }}</div>
          </div>
        </div>
      </div>


    </div>

    <div
        class="drop-area"
        @drop.prevent="handleDrop"
        @dragover.prevent="handleDragOver"
    >
      <p class="drop-text">Datei hier ablegen</p>

    </div>

    <div class="upload-section" v-if="files.length > 0">
      <div class="upload-files" v-for="(file, index) in files" :key="index">{{ file.name }}</div>
      <button class="sql-button" @click="uploadFiles">SQL generieren</button>
    </div>


    <div class="statement-wrapper" v-if="sqlStatements.length > 0">
      <div class="sql-heading">
        <p>SQL Inserts</p>
        <div>
          <button @click="copyAllSQL" class="sql-button">
            Alles kopieren
          </button>
        </div>
      </div>

      <div v-for="(sql, index) in sqlStatements" :key="index" class="sql-statement">
        <p>
          {{ displayShortHex(sql) }}
        </p>
        <div class="image-wrapper">
          <img v-if="filePreviews[getFileNameFromSQL(sql)]" :src="filePreviews[getFileNameFromSQL(sql)]"
               class="preview-image" alt="preview image"/>
        </div>
        <div class="copy-buttons">
          <button @click="copySQL(sql)" class="sql-button-small">
            Zeile kopieren
          </button>
          <button @click="generateHTMLTag(getFileNameFromSQL(sql))" class="sql-button-small">
            HTML Tag
          </button>
        </div>
      </div>
    </div>
  </div>

</template>

<script setup lang="ts">
import {computed, ref} from 'vue';

const showTFiles = ref(false)

function toggleTFiles() {
  showTFiles.value = !showTFiles.value
}

const showTWebsites = ref(false)

function toggleTWebsites() {
  showTWebsites.value = !showTWebsites.value
}

function toggleTableDropdown() {
  showTableDropdown.value = !showTableDropdown.value;
}

const showTableDropdown = ref(false);

const tableOptions = [
  {name: 'spusu.com', tableNumber: '10248'},
  {name: 'spusu.at', tableNumber: '10241'},
  {name: 'spusu.it', tableNumber: '10240'},
  {name: 'helpmobile', tableNumber: '10242'},
  {name: 'eduscho', tableNumber: '10241'},
  {name: 'spusu ebike', tableNumber: '1024138143'},
  {name: 'spusu wein', tableNumber: '1024111314'}
]

const hasVueSite = ref(false);
const selectedTableWeb = ref(tableOptions[0]); // Default to the first option

function selectTable(option) {
  selectedTableWeb.value = option;
  showTableDropdown.value = false;
}

const websiteId = ref(0);
const link = ref('');

// Computed property for the current timestamp
function getCurrentTimestamp(): string {
  const now = new Date();
  return now.toISOString().slice(0, 19).replace('T', ' ');
}

function generateInsertStatements() {
  const tableNumber = selectedTableWeb.value.tableNumber;
  const timestamp = getCurrentTimestamp();

  let sqlStatements = [];

  switch (selectedTableWeb.value) {
    case 'spusu.com':
      if (hasVueSite.value) {
        const sql = `INSERT INTO \`DB_CMS\`.\`t_Websites_${tableNumber}\` (\`nWebsiteId\`, \`strLink\`, \`strVuePath\`) VALUES (${websiteId.value}, '${link.value}', 'mvno_${link.value}');`;
        sqlStatements.push(sql);
      } else {
        const sql1 = `INSERT INTO \`DB_CMS\`.\`t_Websites_${tableNumber}\` (\`nWebsiteId\`, \`strLink\`) VALUES (${websiteId.value}, '${link.value}');`;
        const sql2 = `INSERT INTO \`DB_CMS\`.\`t_WebContent_${tableNumber}\` (\`nWebContentId\`, \`nLanguageId\`, \`strContent\`, \`dtOn\`) VALUES (${websiteId.value}, 1, '<main></main>', '${timestamp}');`;
        sqlStatements.push(sql1, sql2);
      }
      break;
    case 'spusu.it':
      const sql1_it = `INSERT INTO \`DB_CMS\`.\`t_Websites_${tableNumber}\` (\`nWebsiteId\`, \`strLink\`, \`nWebContentId\`) VALUES (${websiteId.value}, '${link.value}', ${websiteId.value});`;
      const sql2_it = `INSERT INTO \`DB_CMS\`.\`t_WebContent_${tableNumber}\` (\`nWebContentId\`, \`nLanguageId\`, \`strContent\`, \`dtOn\`) VALUES (${websiteId.value}, 2, '<main></main>', '${timestamp}');`;
      sqlStatements.push(sql1_it, sql2_it);
      break;
    case 'spusu.at':
    case 'helpmobile':
    case 'eduscho':
    case 'spusu ebike':
    case 'spusu wein':
      const sql1_generic = `INSERT INTO \`DB_CMS\`.\`t_Websites_${tableNumber}\` (\`nWebsiteId\`, \`strLink\`, \`nWebContentId\`) VALUES (${websiteId.value}, '${link.value}', ${websiteId.value});`;
      const sql2_generic = `INSERT INTO \`DB_CMS\`.\`t_WebContent_${tableNumber}\` (\`nWebContentId\`, \`nLanguageId\`, \`strContent\`, \`dtOn\`) VALUES (${websiteId.value}, 1, '<main></main>', '${timestamp}');`;
      sqlStatements.push(sql1_generic, sql2_generic);
      break;
    default:
      break;
  }

  console.log(sqlStatements.join('\n'));
}


const showFolderDropdown = ref(false)
const selectedFolder = ref('spusu.at')
const showLanguageDropdown = ref(false)
const files = ref<File[]>([]);
const sqlStatements = ref<string[]>([])
const selectedLanguageId = ref(0);
const selectedTable = ref('t_Files_10240')
const languageIds = [-1, 0, 1]
const tables = [
  {folder: 'spusu.it', table: 't_Files_10240'},
  {folder: 'spusu.at', table: 't_Files_10241'},
  {folder: 'helpmobile', table: 't_Files_10242'},
  {folder: 'eduscho', table: 't_Files_10244'},
  {folder: 'spusu.com', table: 't_Files_10248'},
  {folder: 'spusu ebike', table: 't_Files_1024138143'},
  {folder: 'spusu wein', table: 't_Files_1024111314'},

]

const fileTypeMap: Record<string, number> = {
  'png': 1,
  'jpg': 2,
  'jpeg': 2,
  'pdf': 5,
  'css': 6,
  'js': 21,
  'svg': 23
}
const filePreviews = ref<{ [key: string]: string }>({});


function selectLanguage(id) {
  selectedLanguageId.value = id;
  showLanguageDropdown.value = false;
}

function toggleLanguageDropdown() {
  showLanguageDropdown.value = !showLanguageDropdown.value;
}


const filteredTables = computed(() => {
  return tables.filter(table => table.folder === selectedFolder.value);
})

function toggleFolderDropdown() {
  showFolderDropdown.value = !showFolderDropdown.value;
}

function selectFolder(folder) {
  selectedFolder.value = folder;
  showFolderDropdown.value = false;
  if (filteredTables.value.length > 0) {
    selectedTable.value = filteredTables.value[0].table;
  }
}

async function generateHTMLTag(filename: string) {
  const formattedString = `/files?f=${filename}`;
  await navigator.clipboard.writeText(formattedString);
  alert('HTML Tag kopiert');
}

function handleDrop(event: DragEvent) {
  const droppedFiles = event.dataTransfer?.files;
  if (droppedFiles) {
    for (let i = 0; i < droppedFiles.length; i++) {
      files.value.push(droppedFiles[i]);
    }
  }
}

function handleDragOver(event: DragEvent) {
  event.preventDefault();
}


async function uploadFiles() {
  for (const file of files.value) {
    const fileContent = await readFileAsHex(file);
    const sqlStatement = generateSQL(file.name, fileContent, selectedLanguageId.value);
    sqlStatements.value.push(sqlStatement);
  }
  files.value = [];
}


function readFileAsHex(file: File): Promise<string> {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      const buffer = e.target?.result;
      const hex = bufferToHex(buffer as ArrayBuffer);
      if (isImageFile(file)) {
        filePreviews.value[file.name] = URL.createObjectURL(file);
      }
      resolve(hex);
    }
    reader.onerror = (e) => reject(e);
    reader.readAsArrayBuffer(file);
  })
}

function isImageFile(file: File): boolean {
  return ['image/png', 'image/jpeg', 'image/jpg', "image/svg"].includes(file.type);
}

function getFileNameFromSQL(sql: string): string {
  const match = /VALUES \('([^']+)/.exec(sql);
  return match ? match[1] : '';
}

function bufferToHex(buffer: ArrayBuffer): string {
  return Array.from(new Uint8Array(buffer))
      .map(b => b.toString(16).padStart(2, '0'))
      .join('');
}

function generateSQL(fileName: string, fileContent: string, languageId: number): string {
  const fileExtension = fileName.split('.').pop()?.toLowerCase() || '';
  const nType = fileTypeMap[fileExtension] || 0;
  return `INSERT INTO \`DB_CMS\`.\`${selectedTable.value}\` (\`strKey\`, \`strFileName\`, \`nLanguageId\`, \`strContent\`, \`nType\`) VALUES ('${fileName}', '${fileName}', ${languageId}, 0x${fileContent}, ${nType});`;
}

async function copyAllSQL() {
  const allSQL = sqlStatements.value.join('\n')
  await navigator.clipboard.writeText(allSQL);
  alert('Alle Statements kopiert')
}

async function copySQL(sql: string) {
  await navigator.clipboard.writeText(sql);
  alert('SQL Zeile kopiert')
}


function displayShortHex(sql: string): string {
  return sql.replace(/(0x[0-9a-fA-F]{5})[0-9a-fA-F]+/, '$1...');
}

</script>

<style>

.select-insert-option {
  width: 100%;
  display: flex;
  justify-content: center;
  gap: 50px;
  margin-bottom: 50px;
}

h1 {
  text-align: center;
  margin: 50px auto;
}

.sql-heading {
  display: flex;
  justify-content: space-between;
  font-weight: bold;
  font-size: 18px;
}

.page-wrapper {
  width: 80%;
  margin: 0 auto;
}

.settings {
  margin-bottom: 20px;
}

.drop-area {
  border: 2px dashed black;
  padding: 20px;
  text-align: center;
  margin: 40px auto;
  width: 800px;
  height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.drop-area:hover {
  transform: scale(1.02);
}

.drop-text {
  font-size: 18px;
  font-weight: bold;
}

.copy-buttons {
  margin-left: 50px;
  display: flex;
  gap: 10px;
}

.sql-button {
  width: 200px;
  border: 2px solid black;
  padding: 10px;
  cursor: pointer;
  align-self: center;
  font-size: 18px;
}

.sql-button:active {
  transform: scale(1.01);
}

.sql-button:hover {
  transform: scale(1.01);
}

.sql-button-small {
  width: 130px;
  border: 2px solid black;
  padding: 10px;
  cursor: pointer;
  align-self: center;
  font-size: 16px;
}

.sql-button-small {
  transform: scale(1.01);
}

.sql-button-small {
  transform: scale(1.01);
}

.image-wrapper {
  width: 140px;
  height: 80px;
}

.preview-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.upload-section {
  width: 860px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.upload-files {
  font-size: 20px;
  font-weight: bold;
}

label {
  font-size: 18px;
  margin-right: 20px;
  font-weight: bold;
}

.settings {
  display: flex;
  width: 100%;
  justify-content: center;
  align-items: center;
  gap: 100px;
}


.dropdown {
  position: relative;
  display: inline-block;
}

.dropdown-selected {
  padding: 8px;
  border: 2px solid black;
  cursor: pointer;
  width: 160px;
  display: flex;
  justify-content: center;
}

.dropdown-options {
  position: absolute;
  background-color: #f9f9f9;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  z-index: 1;
  width: 160px;
  border: 2px solid black;
  padding: 8px;
}

.dropdown-options div {
  padding: 8px;
  cursor: pointer;
}

.dropdown-options div:hover {
  background-color: #f1f1f1;
}

.statement-wrapper {
  border: 2px dashed black;
  display: flex;
  flex-direction: column;
  padding: 40px 50px;
  gap: 10px;
}

.sql-statement {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 18px;
  border: 2px solid black;
  padding: 10px;
  box-sizing: border-box;
  z-index: 20;

}
</style>