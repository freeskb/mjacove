<template>
  <div>
    <input type="file" ref="fileInput1" @change="handleFileChange(1)" />
    <input type="file" ref="fileInput2" @change="handleFileChange(2)" />
    <button @click="uploadExcel">Upload Excel</button>

    <!-- 문자가 포함된 항목을 표시 -->
    <div v-if="filteredData.length > 0">
      <h2>❤️민주❤️</h2>
      <!-- <table>
        <thead>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in filteredData" :key="rowIndex">
          <td v-for="(value, colIndex) in row" :key="colIndex">{{ value }}</td>
        </tr>
        </tbody>
      </table> -->
      <div v-for="(row, rowIndex) in filteredData" :key="rowIndex">
        {{ row }}
      </div>
    </div>
  </div>
</template>

<script>
import * as XLSX from "xlsx";

export default {
  data() {
    return {
      tableData: [[], []],
      filteredData: [],
    };
  },
  methods: {
    handleFileChange(fileIndex) {
      const fileInput = this.$refs[`fileInput${fileIndex}`];
      const file = fileInput.files[0];

      if (file) {
        this.readExcel(file, fileIndex);
      }
    },
    readExcel(file, fileIndex) {
      const reader = new FileReader();
      reader.onload = (e) => {
        const data = e.target.result;
        const workbook = XLSX.read(data, { type: "binary" });
        const sheetName = workbook.SheetNames[0];
        const sheet = workbook.Sheets[sheetName];
        const jsonData = XLSX.utils.sheet_to_json(sheet, { header: 1 });
        this.tableData[fileIndex - 1] = jsonData.map((row) => {
          const rowData = {};
          for (let i = 0; i < row.length; i++) {
            rowData[`Column${i + 1}`] = row[i];
          }
          return rowData;
        });
      };
      reader.readAsBinaryString(file);
    },
    uploadExcel() {
      // 두 개의 파일 내용을 비교하고 문자가 포함된 항목 찾기
      const file1Data = this.tableData[0];
      const file2Data = this.tableData[1];

      this.filteredData = this.findMatchingItems(file1Data, file2Data);

      // 여기에서 서버로 업로드하거나 필요한 작업을 수행할 수 있습니다.
      // 예: axios.post('/api/upload', { data: this.filteredData })
    },
    findMatchingItems(data1, data2) {
      // 문자가 포함된 항목 찾기
      const filteredData = [];
      data1.forEach((item1) => {
        const userName = item1['Column'+23].slice(0, -3) + '***';
        data2.forEach((item2) => {
          const userName2 = item2['Column'+12];
              // 끝에 4글자를 '*'로 치환
          // if (userName2 && userName2.length >= 4) {
          //   userName2 = userName2.slice(0, -4) + '****';
          // }
          if (userName == userName2) {
            filteredData.push(item2['Column'+4]); 
          }
        });
      });

      return filteredData;
    },
  },
};
</script>

<style>
/* 추가적인 스타일링을 적용할 수 있습니다. */
</style>
