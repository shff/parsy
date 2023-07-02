<template>
  <div>
    <select v-model="filetype">
      <option value="txt">txt</option>
      <option value="csv">csv</option>
      <option value="xml">xml</option>
    </select>
    <input type="file" @change="upload" />

    <div class="panel">
      <div class="view">
        <table v-if="filetype === 'txt' || filetype === 'csv'">
          <tr height="30px">
            <th v-for="(field, index) in fields" :key="index">
              {{ field.name }}
            </th>
          </tr>
          <tr v-for="(row, index) in rows" :key="`${index}${row}`">
            <td v-for="(col, index2) in row" :key="index2" class="column">
              <pre>{{ col }}</pre>
            </td>
          </tr>
        </table>
        <div v-else-if="filetype === 'xml'">
          <pre>{{ data }}</pre>
        </div>
      </div>

      <div class="fields">
        <div class="field-list">
          <button @click="newField">+</button>
          <div v-for="(field, index) in fields" :key="index" class="field">
            <div class="field-main">
              <input v-model="field.name" type="text" />
              <input v-model="field.size" type="number" />
            </div>
            <button @click="deleteField(index)">-</button>
            <button @click="moveUp(index)">⬆</button>
            <button @click="moveDown(index)">⬇</button>
          </div>
        </div>

        <div class="json-result">
          <pre>{{ fields }}</pre>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const filetype = ref('txt');
const data = ref('');
const fields = ref([] as { name: string, size: number }[]);

const rows = computed(() => {
  if (filetype.value === 'txt') {
    return data.value.split('\n').map((line) => {
      const row: string[] = [];
      let start = 0;
      fields.value.forEach((field) => {
        row.push(line.substr(start, field.size));
        start += field.size;
      });
      return row;
    })
  } else if (filetype.value === 'csv') {
    return data.value.split('\n').map((row) => row.split(','));
  } else if (filetype.value === 'xml') {
    return data.value;
  }
});

const upload = (e: Event) => {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = (e) => {
    data.value = atob((e.target?.result as string).split(',')[1]);
  }
  reader.readAsDataURL(file);
}

const newField = () => {
  fields.value.push({ name: 'id', size: 10 });
};

const deleteField = (index: number) => {
  fields.value.splice(index, 1);
};

const moveUp = (index: number) => {
  if (index === 0) return;
  const field = fields.value.splice(index, 1)[0];
  fields.value.splice(index - 1, 0, field);
};

const moveDown = (index: number) => {
  if (index === fields.value.length - 1) return;
  const field = fields.value.splice(index, 1)[0];
  fields.value.splice(index + 1, 0, field);
};
</script>

<style scoped>
.panel {
  height: 80vh;
  width: 90vw;
  margin: 16px auto;
  border: 1px solid #333;
  background-color: #222;
  text-align: initial;
  display: flex;
}

.view {
  flex-basis: 70%;
  flex-shrink: 0;
  padding: 8px;
  overflow: auto;
}

table {
  table-layout: fixed;
}

th {
  overflow: hidden;
}

.column {
  border-right: solid 1px #555;
}

.fields {
  flex-grow: 0;
  flex-basis: 30%;
  border-left: 1px solid #333;
  padding: 8px;
  display: flex;
  flex-direction: column;
}

.field-list {
  flex-grow: 2;
  overflow: auto;
}

.field {
  display: flex;
  align-items: center;
  margin: 8px 0;
}

.field-main {
  flex-grow: 2;
}

.field button,
.field input {
  padding: 4px;
  margin-right: 8px;
  border: solid 1px rgba(255, 255, 255, 0.2);
}

.field button {
  width: 24px;
}

.field input[type="number"] {
  width: 40px;
}

.json-result {
  flex-basis: 200px;
  width: 300px;
  margin-top: 16px;
  border: solid 1px #333;
  overflow: scroll;
}

pre {
  margin: 0;
}

th {
  text-align: left;
}
</style>
