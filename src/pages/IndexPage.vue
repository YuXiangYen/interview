<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl" id="addForm">
      <q-form ref="addForm" class="q-mb-xl">
        <q-input
          v-model="tempData.name"
          label="姓名"
          :rules="[(val) => !!val || '姓名不可為空']"
          lazy-rules
        />
        <q-input
          v-model="tempData.age"
          label="年齡"
          :rules="[
            (val) => !!val || '年齡不可為空',
            (val) => val > 0 || '年齡必須大於 0',
          ]"
          lazy-rules
        />
        <q-btn
          color="primary"
          class="q-mt-md"
          :label="isEditing ? '修改' : '新增'"
          @click="addData(tempData.name, tempData.age)"
        />
      </q-form>
      <div>
        <q-btn color="primary" class="q-mt-md" @click="query">查詢</q-btn>
      </div>
      <div class="q-pt-xl">
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
                <div>{{ col.value }}</div>
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
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from "axios";
import { QTableProps } from "quasar";
import { onMounted, ref } from "vue";
import { useToast } from "vue-toast-notification";
import "vue-toast-notification/dist/theme-sugar.css";
const $toast = useToast();

interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([]);

const tableConfig = ref([
  {
    label: "姓名",
    name: "name",
    field: "name",
    align: "left",
  },
  {
    label: "年齡",
    name: "age",
    field: "age",
    align: "left",
  },
]);
const tableButtons = ref([
  {
    label: "編輯",
    icon: "edit",
    status: "edit",
  },
  {
    label: "刪除",
    icon: "delete",
    status: "delete",
  },
]);

const tempData = ref({
  name: "",
  age: "",
  id: "",
});

async function query() {
  try {
    const response = await axios.get("https://dahua.metcfire.com.tw/api/CRUDTest/a");
    blockData.value = response.data;
  } catch (error) {
    console.log(error);
  }
}

const addForm = ref(null);
const isEditing = ref(false);

async function addData(name: string, age: number) {
  if (!addForm.value) return;

  const isValid = await addForm.value.validate();

  if (!isValid) {
    $toast.error("新增失敗，請稍後再試");
    return;
  }

  try {
    const params = ref({});
    if (isEditing.value) {
      params.value = {
        name,
        age: Number(age),
        id: tempData.value.id,
      };
    } else {
      params.value = {
        name,
        age: Number(age),
      };
    }

    const response = await axios.post(
      "https://dahua.metcfire.com.tw/api/CRUDTest",
      params.value
    );

    if (response) {
      if (isEditing.value) {
        isEditing.value = false;
        $toast.success("修改成功");
      } else {
        $toast.success("新增成功");
      }
      query();
    }

    tempData.value.name = "";
    tempData.value.age = "";

    addForm.value.resetValidation();
  } catch (error) {
    $toast.error("新增失敗，請稍後再試");
  }
}

async function handleClickOption(btn, data) {
  // ...

  if (btn.status === "edit") {
    isEditing.value = true;
    tempData.value.name = data.name;
    tempData.value.age = data.age;
  } else if (btn.status === "delete") {
    if (confirm(`確定要刪除${data.name}嗎?`)) {
      try {
        const response = await axios.delete(
          `https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`
        );
        alert("刪除成功");
        query();
      } catch (error) {
        $toast.error("刪除失敗，請稍後再試");
      }
    }
  }
}

onMounted(async () => {
  await query();
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
