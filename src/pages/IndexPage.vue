<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <!-- 表單 -->
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" outlined />
        <q-input v-model="tempData.age" label="年齡" type="number" outlined />
        <q-btn
          color="primary"
          class="q-mt-md"
          @click="tempData.id ? saveEdit() : handleAdd()"
        >
          {{ tempData.id ? '保存修改' : '新增' }}
        </q-btn>
      </div>

      <!-- 資料表格 -->
      <q-table
        flat
        bordered
        :key="forceRenderKey"
        :rows="blockData"
        :columns="tableConfig"
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
            <q-th>操作</q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td v-for="col in props.cols" :key="col.name" :props="props">
              <div>{{ props.row[col.name] }}</div>
            </q-td>
            <q-td class="text-right" auto-width>
              <q-btn
                v-for="btn in tableButtons"
                :key="btn.label"
                :icon="btn.icon"
                @click="
                  btn.status === 'edit'
                    ? handleEdit(props.row)
                    : handleDelete(props.row.id)
                "
                dense
                flat
              >
                <q-tooltip>{{ btn.label }}</q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>

        <template v-slot:no-data>
          <div class="full-width row flex-center items-center text-primary">
            無相關資料
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'; // 移除未使用的 reactive
import axios from 'axios';
import type { QTableProps } from 'quasar';

// 表格數據和設定
const blockData = ref<{ id: string; name: string; age: number }[]>([]); // 確保 id 是 String
const tableConfig = ref<QTableProps['columns']>([
  { label: '姓名', name: 'name', field: 'name', align: 'left' }, // 映射 name
  { label: '年齡', name: 'age', field: 'age', align: 'left' }, // 映射 age
]);

const tableButtons = ref([
  { label: '編輯', icon: 'edit', status: 'edit' },
  { label: '刪除', icon: 'delete', status: 'delete' },
]);

// 表單數據
const tempData = ref<{ id?: string; name: string; age: number }>({
  name: '',
  age: 0,
});

// 引入一個強制重繪的計數器
const forceRenderKey = ref(0); // 用於強制觸發 q-table 重繪

// 新增功能
async function handleAdd() {
  if (!tempData.value.name || tempData.value.age <= 0) {
    alert('姓名與年齡不得為空，且年齡必須為正數');
    return;
  }
  try {
    const response = await axios.post(
      'https://dahua.metcfire.com.tw/api/CRUDTest',
      tempData.value
    );
    const newData = response.data;

    // 透過創建新陣列觸發響應
    blockData.value = [...blockData.value, newData]; // 將新資料加入陣列
    tempData.value = { name: '', age: 0 }; // 清空表單

    await fetchData(); // 重新整理資料
  } catch (error) {
    console.error('新增失敗:', error);
  }
}


// 編輯功能
function handleEdit(data: { id: string; name: string; age: number }) {
  tempData.value = { ...data }; // 將選中的行數據填充到表單中
}

// 保存修改功能
async function saveEdit() {
  if (!tempData.value.id) {
    alert('無法修改，缺少 ID');
    return;
  }
  try {
    const response = await axios.patch(
      'https://dahua.metcfire.com.tw/api/CRUDTest',
      tempData.value
    );
    const updatedData = response.data;

    // 更新資料並創建新陣列觸發響應
    const index = blockData.value.findIndex((row) => row.id === updatedData.id);
    if (index !== -1) {
      // 使用 splice 方法更新陣列
      blockData.value.splice(index, 1, updatedData); // 替換舊數據
    }
    tempData.value = { name: '', age: 0 }; // 清空表單

    await fetchData(); // 重新整理資料
  } catch (error) {
    console.error('修改失敗:', error);
  }
}


// 刪除功能
async function handleDelete(id: string) {
  if (!confirm('您確定要刪除此資料嗎？')) return;
  try {
    await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`);
    blockData.value = blockData.value.filter((row) => row.id !== id);
    forceRenderKey.value++; // 強制重新渲染
  } catch (error) {
    console.error('刪除失敗', error);
  }
}

// 查詢功能
async function fetchData() {
  try {
    const response = await axios.get(
      'https://dahua.metcfire.com.tw/api/CRUDTest/a'
    );
    blockData.value = response.data;
    forceRenderKey.value++; // 強制重新渲染
  } catch (error) {
    console.error('獲取資料失敗', error);
  }
}

// 頁面加載時調用查詢功能
onMounted(() => {
  fetchData();
});
</script>

<style scoped lang="scss">
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
