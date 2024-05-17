<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="addData">新增</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ props.row[col.name] }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref, onMounted } from 'vue';

interface btnType {
  label: string;
  icon: string;
  status: string;
}

const blockData = ref([]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);

const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: '',
  age: '',
});
//取得資料
function fetchData() {
  axios
    .get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
    .then((response) => {
      blockData.value = response.data;
    })
    .catch((error) => {
      console.error('Error fetching data:', error);
    });
}
//新增資料+判斷姓名年紀不得為空值
function addData() {
  if (!tempData.value || !tempData.value.age) {
    alert('姓名與年紀不得為空值');
    return;
  } else {
    axios
      .post('https://dahua.metcfire.com.tw/api/CRUDTest', tempData.value)
      .then(() => {
        fetchData();
        tempData.value.name = '';
        tempData.value.age = '';
      })
      .catch((error) => {
        console.error('Error adding data:', error);
      });
  }
}

//更新資料
function updateData(id, newData) {
  axios
    .patch(`https://dahua.metcfire.com.tw/api/CRUDTest`, { id, ...newData })
    .then(() => {
      fetchData();
    })
    .catch((error) => {
      console.error('Error updating data:', error);
    });
}

//刪除資料
function deleteData(id) {
  axios
    .delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`)
    .then(() => {
      fetchData();
    })
    .catch((error) => {
      console.error('Error deleting data:', error);
    });
}

//更新資料
function handleClickOption(btn, data) {
  if (btn.status === 'edit') {
    const newName = prompt('請輸入新名字:', data.name);
    const newAge = prompt('請輸入新年齡:', data.age);
    if (newName !== null && newAge !== null) {
      updateData(data.id, { name: newName, age: newAge });
    }
  } else if (btn.status === 'delete') {
    if (confirm('是否確定刪除該筆資料?')) {
      deleteData(data.id);
    }
  }
}

onMounted(() => {
  fetchData();
});
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
