<script setup lang="ts">
import { onMounted, ref, watch } from 'vue'
import axios from "@/utils/httpclient";
import rank_store from './store'

// 定义响应式变量
const tableData = ref([]) // 不再是响应式对象
const checkboxValues = ref<[string[], string[]]>([[], []]);
const currentPage = ref(1); // 追踪当前页码
const pageSize = ref(12)
const checkAllParamsRange = ref(false);
const checkAllCheckboxData = ref(false);
const isIndeterminateParamsRange = ref(false);
const isIndeterminateCheckboxData = ref(false);
const paramsRange = ref([
  { id: 1, label: "6B", value: "6B" },
  { id: 2, label: "7B", value: "7B" },
  { id: 3, label: "13B", value: "13B" },
  { id: 4, label: "14B", value: "14B" },
  { id: 5, label: "34B", value: "34B" },
  { id: 6, label: "72B", value: "72B" },
]);
const checkboxData = ref([
  { id: 1, label: "合规性", value: "合规性" },
  { id: 2, label: "公平性", value: "公平性" },
  { id: 3, label: "知识产权", value: "知识产权" },
  { id: 4, label: "隐私保护", value: "隐私保护" },
  { id: 5, label: "可信度", value: "可信度" },
]);
const dynamicColumns = ref<any[]>([]);
const loading = ref(true);


// 获取报告列表
const fetch_report = () => {
  loading.value = true; // 在请求开始时设置 loading 为 true
  axios.post('/vcis11/get_leaderboard_detail', {}, {})
    .then(response => {
      loading.value = false;
      if (response.data && response.data.data) {
        localStorage.setItem('reportData', JSON.stringify(response.data));
        updateTableData(response.data);
      } else {
        console.error('API 返回的数据格式不正确');
        // 使用默认数据或显示错误消息
        updateTableData({ data: [] });
      }
    })
    .catch(error => {
      console.error('发生错误:', error);
      loading.value = false;
      // 使用默认数据或显示错误消息
      updateTableData({ data: [] });
    });
};
// 修改 updateTableData 函数
const updateTableData = (data) => {
  const cate = checkboxValues.value[0];
  const modelParams = checkboxValues.value[1];

  if (!data.data || !Array.isArray(data.data)) {
    console.error('数据格式不正确');
    tableData.value = [];
    return;
  }

  const resultList = data.data.filter(item => !item.参数量 || cate.includes(item.参数量));

  tableData.value = resultList.map(item => {
    const newItem = { ...item };
    for (const oneKey of modelParams) {
      if (!newItem[oneKey]) {
        delete newItem[oneKey];
      }
    }
    return newItem;
  });

  // 更新动态列
  dynamicColumns.value = checkboxValues.value[1].map(value => ({
    prop: value,
    label: value,
    width: 167,
    fixed: 'right',
    sortable: true,
  }));

  // 检查是否有数据
  if (tableData.value.length === 0) {
    console.log('没有符合条件的数据');
    // 可以在这里添加一些用户提示
  }
};

// 在 setup 中添加
onMounted(() => {
  fetchReportFromLocalStorage();
});// 从本地缓存中取数据
// 修改 fetchReportFromLocalStorage 函数
const fetchReportFromLocalStorage = () => {
  const reportDataJSON = localStorage.getItem('reportData');
  if (reportDataJSON) {
    try {
      const reportData = JSON.parse(reportDataJSON);
      updateTableData(reportData);
    } catch (error) {
      console.error('解析报告数据时发生错误:', error);
      fetch_report(); // 如果本地数据无效，从服务器获取
    }
  } else {
    fetch_report(); // 如果没有本地数据，从服务器获取
  }
};
// 分页处理
const paginateTableData = () => {
  const startIndex = (currentPage.value - 1) * 12;
  const endIndex = startIndex + 12;
  return tableData.value.slice(startIndex, endIndex);
};

// 处理全选操作
const handleCheckAllParamsRangeChange = (val: boolean) => {
  checkboxValues.value[0] = val ? paramsRange.value.map(item => item.value) : [];
  isIndeterminateParamsRange.value = false;
};

const handleCheckAllCheckboxDataChange = (val: boolean) => {
  checkboxValues.value[1] = val ? checkboxData.value.map(item => item.value) : [];
  isIndeterminateCheckboxData.value = false;
};

// 处理单个维度选择
const handleCheckModelChange = (value: string[]) => {
  const checkedCountParamsRange = value.filter(item => paramsRange.value.some(param => param.value === item)).length;
  const checkedCountCheckboxData = value.filter(item => checkboxData.value.some(checkbox => checkbox.value === item)).length;

  checkAllParamsRange.value = checkedCountParamsRange === paramsRange.value.length;
  isIndeterminateParamsRange.value = checkedCountParamsRange > 0 && checkedCountParamsRange < paramsRange.value.length;

  checkAllCheckboxData.value = checkedCountCheckboxData === checkboxData.value.length;
  isIndeterminateCheckboxData.value = checkedCountCheckboxData > 0 && checkedCountCheckboxData < checkboxData.value.length;
};

// 处理页码变更
const handlePageChange = (val: number) => {
  currentPage.value = val;
  //fetch_report();
};

const getRowIndex = (index: number) => {
  return (currentPage.value - 1) * pageSize.value + index + 1;
};

const resetFilters = () => {
  // 清除模型规模过滤器的选中状态
  checkboxValues.value[0] = [];
  checkAllParamsRange.value = false;
  isIndeterminateParamsRange.value = false;

  // 清除显示维度过滤器的选中状态
  checkboxValues.value[1] = [];
  checkAllCheckboxData.value = false;
  isIndeterminateCheckboxData.value = false;

  // 重新获取报告列表
  fetch_report();

  // 重置动态列为初始状态
  dynamicColumns.value = [];
};
const queryModels = () => {
  fetchReportFromLocalStorage();
  const filteredData = responseData.data.filter(row => {
    // 检查模型规模是否符合用户选择
    if (checkboxValues.value[0].length > 0 && !checkboxValues.value[0].includes(row.模型规模)) {
      return false;
    }

    // 检查显示维度是否符合用户选择
    if (checkboxValues.value[1].length > 0 && !checkboxValues.value[1].some(value => row[value])) {
      return false;
    }

    return true;
  });

  // 更新表格数据
  tableData.value = filteredData;

  // 生成动态列
  dynamicColumns.value = checkboxValues.value[1].map(value => ({
    prop: value,
    label: value,
    width: 125, // 可以根据需要设置列宽
    fixed: 'right', // 可以根据需要设置是否固定列
    sortable: true, // 可以根据需要设置是否可排序
  }));
};




// 生命周期钩子
onMounted(fetch_report);

watch(checkboxValues, () => {
  queryModels();
});
</script>

<template>
  <div>
    <el-container>
      <el-main>
        <div class="title-container">
          <h1 class="title-item">排行榜</h1>
        </div>
        <div class="rank-filter-container">
          <div class="para-container">
            <!-- 查询模型规模 -->
            <div class="para-item">
              <h1 class="para-item-title">查询模型规模：</h1>
              <div class="checkbox-item">
                <el-row class="inline-checkbox">
                  <el-checkbox :indeterminate="isIndeterminateParamsRange" v-model="checkAllParamsRange"
                    @change="handleCheckAllParamsRangeChange">全选</el-checkbox>
                  <el-checkbox-group v-model="checkboxValues[0]" @change="handleCheckModelChange">
                    <el-checkbox v-for="item in paramsRange" :key="item.id" :label="item.label" :value="item.value">
                      {{ item.label }}
                    </el-checkbox>
                  </el-checkbox-group>
                </el-row>
              </div>
            </div>
            <!-- 显示维度 -->
            <div class="para-item">
              <h1 class="para-item-title">显示维度：</h1>
              <div class="checkbox-item">
                <el-row class="inline-checkbox">
                  <el-checkbox :indeterminate="isIndeterminateCheckboxData" v-model="checkAllCheckboxData"
                    @change="handleCheckAllCheckboxDataChange">全选</el-checkbox>
                  <el-checkbox-group v-model="checkboxValues[1]" @change="handleCheckModelChange">
                    <el-checkbox v-for="item in checkboxData" :key="item.id" :label="item.label" :value="item.value">
                      {{ item.label }}
                    </el-checkbox>
                  </el-checkbox-group>
                </el-row>
              </div>
            </div>
            <!-- 查询和重置按钮 -->
            <div class="button-container">
              <el-button class="button-find-item" type="primary" @click="queryModels">查询</el-button>
              <el-button class="button-reset-item" color="#CED0D8" plain @click="resetFilters">重置</el-button>
            </div>
          </div>
        </div>
        <!-- 表格 -->
        <div class="table-container">
          <el-table :data="paginateTableData()" highlight-current-row v-loading="loading"
            :header-cell-style="{ backgroundColor: '#F5F6F7', color: '#41475E', 'font-size': '14px', 'font-weight': '600', 'line-height': '24px' }"
            :cell-style="{ 'height': '56px', 'line-height': '24px' }" >
            <el-table-column label="序号" width="88" fixed>
              <template v-slot="{ row, $index }">
                <span>{{ getRowIndex($index) }}</span>
              </template>
            </el-table-column>
            <el-table-column prop="模型" label="模型" width="120" fixed></el-table-column>
            <el-table-column prop="参数量" label="模型规模" width="120" fixed sortable></el-table-column>
            <el-table-column prop="发布者" label="发布者" width="120" fixed></el-table-column>
            <el-table-column prop="发布日期" label="发布日期" width="184" fixed sortable></el-table-column>
            <el-table-column prop="国内/国外模型" label="国内/国外" width="95" fixed></el-table-column>
            <el-table-column prop="综合" label="综合" width="163" fixed sortable></el-table-column>
            <el-table-column v-for="column in dynamicColumns" :key="column.prop" :prop="column.prop"
              :label="column.label" :width="column.width" :fixed="column.fixed"
              :sortable="column.sortable"></el-table-column>

            <!-- 
            <el-table-column prop="合规性" label="合规性" width="167" fixed sortable></el-table-column>
            <el-table-column prop="公平性" label="公平性" width="167" fixed sortable></el-table-column>
            <el-table-column prop="知识产权" label="知识产权" width="167" fixed sortable></el-table-column> -->
          </el-table>
        </div>
        <div class="pagination-container">
          <el-pagination v-model:current-page="currentPage" v-model:page-size="pageSize" :page-sizes="[12, 24, 36, 48]"
            background layout="total, prev, pager, next, sizes, jumper" :total="tableData.length"
            @current-change="handlePageChange" />
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<style scoped>
.rank-container {
  align-items: center;
}

.title-container {
  width: 120px;
  height: 30px;
  margin-top: 30px;
  margin-left: 70px;
}

.title-item {
  font-weight: 500;
  font-size: 30px;
  color: #121127;
  line-height: 30px;
  text-align: left;
}

.rank-filter-container {
  display: flex;
  margin-top: 32px;
  margin-bottom: 29px;
}

.table-container {
  width: auto;
  margin-left: 70px;
  margin-right: auto;
}

.para-item-title {
  font-weight: 400;
  font-size: 14px;
  color: #41475E;
  line-height: 22px;
  text-align: left;
  height: 22px;
  width: 98px;
}

.para-item {
  display: flex;
  align-items: center;
  margin-left: 70px;
  grid-row-gap: 25px;
}

.checkbox-item {
  margin-left: 24px;
}

.checkbox-label {
  font-weight: 400;
  font-size: 14px;
  color: #121127;
  line-height: 22px;
  text-align: left;
}

.table-row-item {
  width: 28px;
  height: 24px;
  font-weight: 600;
  font-size: 14px;
  color: #41475E;
  line-height: 24px;
  text-align: left;
}

.button-container {
  margin-top: 25px;
  gap: 16px;
  margin-left: 190px;
}

.button-find-item {
  width: 88px;
  height: 32px;
  font-size: 14px;
  line-height: 22px;
  font-weight: 400;
  color: #FFFFFF;

}

.button-reset-item {
  width: 88px;
  height: 32px;
  border-radius: 4px 4px 4px 4px;
  font-size: 14px;
  line-height: 22px;
  font-weight: 400;
  color: #040B29;
}

.inline-checkbox {
  display: flex;
  flex-direction: row;
  gap: 24px;
}

.pagination-container {
  margin-top: 28px;
  margin-right: 80px;
  display: flex;
  justify-content: flex-end;
}

:deep(.el-table .descending .sort-caret.descending) {
  border-top-color: #121127;
}

:deep(.el-table .ascending .sort-caret.ascending) {
  border-bottom-color: #121127;
}
</style>
