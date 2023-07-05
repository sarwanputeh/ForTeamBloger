<template>
  <div class="container">
    <div class="Header-text2" style="margin-left: 6px">
     
      Criteria
    </div>
    <br /><br />
    <div class="search-box">
      <input  style="width: 300px;" class="input is-info" v-model="searchDocumentName" type="text" placeholder="Search Document Name" />
      <input  style="width: 300px; margin-left: 6px;" class="input is-info" v-model="SearchSystemName" type="text" placeholder="Search System Name" />
     
    </div>
    <br />
    <div class="Header-text2" style="margin-left: 6px"> Result</div>
    <br />
    <div class="table-container">
      <table class="table is-fullwidth">
        <thead>
          <tr>
            <th>No</th>
            <th>DocumentName</th>
            <th>Sys.</th>
            <th>Customer Sign</th>
            <th>TOLC Sign</th>
            <th>Folder</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="document in paginatedDocuments" :key="document.documentTypeCode">
            <td>{{ document.documentTypeCode }}</td>
            <td>{{ document.documentTypeName }}</td>
            <td>{{ document.systemCode }}</td>
            <td>{{ document.folderName }}</td>
           
          </tr>
        </tbody>
      </table>
    </div>
    <nav class="pagination is-centered">
      <ul class="pagination-list">
        <li>
          <button
            @click="changePage(currentPage - 1)"
            class="pagination-link"
            :disabled="currentPage === 1"
          >
            Prev
          </button>
        </li>
        <li v-for="page in paginationRange" :key="page">
          <button
            @click="changePage(page)"
            class="pagination-link"
            :class="{ 'is-current': page === currentPage }"
          >
            {{ page }}
          </button>
        </li>
        <li>
          <button
            @click="changePage(currentPage + 1)"
            class="pagination-link"
            :disabled="currentPage === totalPages"
          >
            Next
          </button>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import axios from "axios";


export default {
  components: {
   
  },
  data() {
    return {
      documents: [],
      pageSize: 10,
      currentPage: 1,
      maxPaginationButtons: 5,
      searchDocumentName: "",
      SearchSystemName: "",
    };
  },
  created() {
    let apiURL = "https://localhost:7083/api/DocumentCustodySetting";

    axios
      .get(apiURL)
      .then((res) => {
        this.documents = res.data;
      })
      .catch((error) => {
        console.log(error);
      });
  },
  // Method จะเรียก Function ทุกครั้งที่ Component re-render ใหม่ ซึ่งต่างจาก Computed จะไม่เรียก Function เมื่อ ข้อมูลไม่เปลี่ยน 
  methods: {
    
    changePage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
      }
    },
    resetSearch() {
      this.searchDocumentName = "";
      this.SearchSystemName = "";
    },
  },
//   เราจะใช้เมื่อต้องการแก้ไขข้อมูลบางตัวซึ่งผูกมัดกับข้อมูลอื่น ๆ
  computed: {
    totalItems() {
      return this.documents.length;
    },
    totalPages() {
      return Math.ceil(this.totalItems / this.pageSize);
    },
    paginationRange() {
      const start = Math.max(1, this.currentPage - Math.floor(this.maxPaginationButtons / 2));
      const end = Math.min(start + this.maxPaginationButtons - 1, this.totalPages);

      return Array.from({ length: end - start + 1 }, (_, i) => start + i);
    },
    paginatedDocuments() {
      //ผมประกาศ filteredDocuments เพื่อเก็บ  this.documents ซึ่งก็คือ Data ที่ดึง API มาเก็บ
      let filteredDocuments = this.documents;

      if (this.searchDocumentName) {
        filteredDocuments = filteredDocuments.filter((document) =>
          document.documentTypeName.toLowerCase().includes(this.searchDocumentName.toLowerCase())
        );
      }

      if (this.SearchSystemName) {
        filteredDocuments = filteredDocuments.filter((document) =>
          document.systemCode.toLowerCase().includes(this.SearchSystemName.toLowerCase())
        );
      }

      const startIndex = (this.currentPage - 1) * this.pageSize;
      const endIndex = startIndex + this.pageSize;

      return filteredDocuments
        .slice(startIndex, endIndex)
        .map((document) => ({
          ...document,
          // customerSign: document.customerSign === 1,
          // tolcSign: document.tolcSign === 1,
        }));
    },
  },
};
</script>

<style scoped>
.container {
  margin-top: 2rem;
}

.table-container {
  overflow-x: auto;
}

.table {
  width: 100%;
  border-collapse: collapse;
}

.table th,
.table td {
  padding: 8px;
  text-align: left;
  border-bottom: 1px solid #f0f0f0;
}

.table th {
  background-color: #f5f5f5;
  font-weight: 500;
}

.pagination {
  margin-top: 2rem;
  justify-content: center;
}

.pagination-list {
  display: flex;
  justify-content: center;
}

.pagination-link {
  padding: 0.5rem;
  border: 1px solid #ccc;
  margin-right: 0.25rem;
  cursor: pointer;
}

.pagination-link.is-current {
  background-color: #007bff;  
  color: #fff;
  border-color: #007bff;
}
</style>
