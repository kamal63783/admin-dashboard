<template>
  <div class="data">
    <!-- Search Bar -->
    <input
      v-model="searchQuery"
      type="text"
      placeholder="Search..."
      class="search-bar"
      @input="performSearch"
      @keyup.enter="performSearch"
    />
    <!-- Delete Selected Button -->
    <div class="delete-selected-container">
      <button
        class="delete-selected-btn"
        :disabled="!isAnyRowSelected"
        @click="deleteSelected"
      >
        <i class="fa-solid fa-trash"></i>
      </button>
    </div>

    <!-- Data Table -->
    <div v-if="data.length > 0" class="data-table-container">
      <table class="data-table">
        <thead>
          <!-- Table Headers -->
          <tr>
            <th>
              <!-- Select/Deselect All Rows -->
              <input
                type="checkbox"
                :checked="areAllRowsSelectedOnPage"
                @change="selectAllRowsOnPage"
              />
            </th>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <!-- Table Rows -->
          <tr
            v-for="(entry, index) in paginatedData"
            :key="index"
            :class="{ selected: entry.isSelected }"
          >
            <!-- Checkbox for Individual Row Selection -->
            <td>
              <input
                v-model="entry.isSelected"
                type="checkbox"
                :checked="isSelected(entry)"
                @change="selectRow(entry)"
              />
            </td>
            <!-- Data Cells -->
            <td>{{ entry.id }}</td>
            <td v-if="!entry.editing">{{ entry.name }}</td>
            <!-- Editing Fields -->
            <td v-else-if="entry.editing">
              <input v-model="entry.name" type="text" />
            </td>
            <td v-if="!entry.editing">{{ entry.email }}</td>
            <td v-else-if="entry.editing">
              <input v-model="entry.email" type="email" />
            </td>
            <td v-if="!entry.editing">{{ entry.role }}</td>
            <td v-else-if="entry.editing">
              <input v-model="entry.role" type="text" />
            </td>
            <!-- Edit, Save, Delete Buttons -->
            <td>
              <button
                v-if="!entry.editing && !entry.changesSaved"
                class="icon-edit"
                @click="toggleEditMode(entry)"
              >
                <i class="fa-regular fa-pen-to-square"></i>
              </button>
              <!-- Cancel and Save Changes -->
              <button
                v-if="entry.editing"
                class="icon-cancel"
                @click="cancelChanges(entry)"
              >
                <i class="fas fa-times"></i>
              </button>
              <button
                v-if="entry.editing"
                class="icon-save"
                @click="saveChanges(entry)"
              >
                <i class="fas fa-check"></i>
              </button>
              <!-- Delete Entry -->
              <button
                v-if="!entry.editing && !entry.changesSaved"
                class="icon-delete"
                @click="deleteEntry(entry)"
              >
                <i class="fa-solid fa-trash"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <div v-else>
      <p class="error-msg">No results found.</p>
    </div>

    <!-- Selected Data Count -->
    <div class="selected-count">
      {{ selectedData.length }} of {{ data.length }} row(s) selected
    </div>

    <!-- Pagination controls -->
    <div class="pagination-container">
      <div class="pagination">
        <!-- Page information -->
        <span v-if="totalPages > 0">
          Page {{ currentPage }} of {{ totalPages }}
        </span>
        <!-- Navigation Buttons -->
        <button
          :disabled="currentPage === 1 || totalPages === 0"
          @click="firstPage"
        >
          &lt;&lt;
        </button>
        <button
          :disabled="currentPage === 1 || totalPages === 0"
          @click="prevPage"
        >
          &lt;
        </button>
        <button v-if="displayedPages[0] > 1" @click="firstPage">1</button>
        <span v-if="displayedPages[0] > 2">...</span>
        <button
          v-for="page in displayedPages"
          :key="page"
          :class="{ 'selected-page': page === currentPage }"
          @click="setPage(page)"
        >
          {{ page }}
        </button>
        <span v-if="displayedPages[displayedPages.length - 1] < totalPages - 1"
          >...</span
        >
        <button
          v-if="displayedPages[displayedPages.length - 1] < totalPages"
          @click="lastPage"
        >
          {{ totalPages }}
        </button>
        <button
          :disabled="currentPage === totalPages || totalPages === 0"
          @click="nextPage"
        >
          >
        </button>
        <button
          :disabled="currentPage === totalPages || totalPages === 0"
          @click="lastPage"
        >
          >>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import '@fortawesome/fontawesome-free/css/all.css'

export default {
  data() {
    return {
      searchQuery: '',
      data: [],
      selectedData: [],
      tempData: [],
      currentPage: 1,
      itemsPerPage: 10,
      maxDisplayedPages: 5,
    }
  },
  computed: {
    isAnyRowSelected() {
      return this.selectedData.length > 0
    },
    areAllRowsSelectedOnPage() {
      if (this.data.length === 0) {
        return false
      }
      const startIndex = (this.currentPage - 1) * this.itemsPerPage
      const endIndex = this.currentPage * this.itemsPerPage

      for (let i = startIndex; i < endIndex && i < this.data.length; i++) {
        if (!this.isSelected(this.data[i])) {
          return false
        }
      }
      return true
    },
    paginatedData() {
      const startIndex = (this.currentPage - 1) * this.itemsPerPage
      const endIndex = this.currentPage * this.itemsPerPage
      return this.data.slice(startIndex, endIndex)
    },
    totalPages() {
      return Math.ceil(this.data.length / this.itemsPerPage)
    },
    displayedPages() {
      const startPage = Math.max(
        2,
        this.currentPage - Math.floor(this.maxDisplayedPages / 2)
      )
      const endPage = Math.min(
        this.totalPages - 1,
        startPage + this.maxDisplayedPages - 1
      )

      const pages = []
      for (let i = startPage; i <= endPage; i++) {
        pages.push(i)
      }
      return pages
    },
  },
  created() {
    this.fetchData()
  },
  mounted() {
    document.title = 'Admin Dashboard'
  },
  methods: {
    async fetchData() {
      try {
        const response = await axios.get(
          'https://geektrust.s3-ap-southeast-1.amazonaws.com/adminui-problem/members.json'
        )
        this.data = response.data.map((entry) => ({
          ...entry,
          editing: false,
        }))
        this.tempData = JSON.parse(JSON.stringify(this.data))
      } catch (error) {}
    },
    fetchTempData() {
      try {
        this.data = this.tempData.map((entry) => ({
          ...entry,
          editing: false,
        }))
      } catch (error) {}
    },
    toggleEditMode(entry) {
      this.data.forEach((item) => {
        if (item !== entry) {
          item.editing = false
          item.changesSaved = false
        }
      })

      entry.editing = !entry.editing
      entry.changesSaved = false
    },
    saveChanges(entry) {
      entry.editing = false
      const index = this.tempData.findIndex((item) => item.id === entry.id)

      if (index !== -1) {
        this.$set(this.tempData, index, {
          ...this.tempData[index],
          name: entry.name,
          email: entry.email,
          role: entry.role,
        })
      }
    },

    cancelChanges(entry) {
      entry.editing = false
      entry.changesSaved = false
    },
    deleteEntry(entry) {
      const indexInTempData = this.tempData.findIndex(
        (item) => item.id === entry.id
      )

      if (indexInTempData !== -1) {
        this.tempData.splice(indexInTempData, 1)
      }

      const indexInData = this.data.indexOf(entry)
      if (indexInData !== -1) {
        this.data.splice(indexInData, 1)
      }

      const currentPageDataStartIndex =
        (this.currentPage - 1) * this.itemsPerPage

      if (this.data.length === 0 && this.currentPage > 1) {
        this.currentPage--
      }

      if (
        this.data.length <= currentPageDataStartIndex &&
        this.currentPage > 1
      ) {
        this.currentPage = 1
      }
    },

    deleteSelected() {
      this.selectedData.forEach((selectedEntry) => {
        // Remove from tempData
        const indexInTempData = this.tempData.findIndex(
          (entry) => entry.id === selectedEntry.id
        )
        if (indexInTempData !== -1) {
          this.tempData.splice(indexInTempData, 1)
        }

        // Remove from data
        const indexInData = this.data.findIndex(
          (entry) => entry.id === selectedEntry.id
        )
        if (indexInData !== -1) {
          this.data.splice(indexInData, 1)
        }
      })

      this.selectedData = []
    },

    selectEntry(entry, isSelected) {
      const index = this.data.indexOf(entry)

      if (index !== -1) {
        this.$set(this.data[index], 'isSelected', isSelected)

        if (isSelected) {
          this.selectedData.push(entry)
        } else {
          const selectedIndex = this.selectedData.indexOf(entry)
          if (selectedIndex !== -1) {
            this.selectedData.splice(selectedIndex, 1)
          }
        }
      }

      const headerCheckbox = document.querySelector(
        '.data-table th input[type="checkbox"]'
      )
      if (headerCheckbox) {
        const checkedCheckboxes = document.querySelectorAll(
          '.data-table tbody tr input[type="checkbox"]:checked'
        )
        headerCheckbox.checked =
          checkedCheckboxes.length === this.paginatedData.length
        headerCheckbox.indeterminate =
          checkedCheckboxes.length > 0 &&
          checkedCheckboxes.length < this.paginatedData.length
      }
    },

    performSearch() {
      const query = this.searchQuery.toLowerCase().trim()
      if (!query) {
        this.fetchTempData()
        return
      }

      const selectedIds = this.selectedData.map((entry) => entry.id)

      this.data = this.tempData.filter((entry) => {
        return (
          entry.id.toLowerCase().includes(query) ||
          entry.name.toLowerCase().includes(query) ||
          entry.email.toLowerCase().includes(query) ||
          entry.role.toLowerCase().includes(query)
        )
      })

      this.data.forEach((entry) => {
        this.$set(entry, 'isSelected', selectedIds.includes(entry.id))
      })

      if (this.currentPage > this.totalPages) {
        this.currentPage = 1
      }
    },

    setPage(pageNumber) {
      this.currentPage = pageNumber
      this.updateSelectedData()
      const headerCheckbox = document.querySelector(
        '.data-table th input[type="checkbox"]'
      )
      if (headerCheckbox) {
        headerCheckbox.checked = false
        headerCheckbox.indeterminate = false
      }
    },
    updateSelectedData() {
      this.selectedData = this.data.filter((entry) => entry.isSelected)
    },

    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--
      }
    },

    firstPage() {
      if (this.currentPage !== 1) {
        this.currentPage = 1
      }
    },

    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++
      }
    },

    lastPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage = this.totalPages
      }
    },

    selectAllRowsOnPage() {
      const startIndex = (this.currentPage - 1) * this.itemsPerPage
      const endIndex = Math.min(
        this.currentPage * this.itemsPerPage,
        this.data.length
      )
      const areAllSelected = this.areAllRowsSelectedOnPage

      for (let i = startIndex; i < endIndex; i++) {
        const currentEntry = this.data[i]
        const isSelected = this.selectedData.includes(currentEntry)

        if (!areAllSelected && !isSelected) {
          this.selectEntry(currentEntry, true)
        } else if (areAllSelected && isSelected) {
          this.selectEntry(currentEntry, false)
        }
      }

      this.updateHeaderCheckboxState()
    },

    updateHeaderCheckboxState() {
      const areAllSelected = this.areAllRowsSelectedOnPage
      const headerCheckbox = document.querySelector(
        '.data-table th input[type="checkbox"]'
      )

      if (headerCheckbox) {
        headerCheckbox.checked = areAllSelected
        headerCheckbox.indeterminate =
          !areAllSelected && this.selectedData.length > 0
      }
    },

    selectRow(entry) {
      const isSelected = this.isSelected(entry)
      this.selectEntry(entry, !isSelected)
    },

    isSelected(entry) {
      return this.selectedData.includes(entry)
    },
  },
}
</script>

<style scoped>
.data {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
    Ubuntu, Cantarell, 'Helvetica Neue', Helvetica, Arial, sans-serif;
  position: relative;
}

.search-bar {
  margin-top: 30px;
  margin-left: 20px;
  margin-bottom: 20px;
  padding: 6px;
  border: 2px solid #ccc;
  border-radius: 6px;
  width: 425px;
}

.delete-selected-container {
  position: fixed;
  top: 20px;
  right: 25px;
  margin-top: 10px;
}

.delete-selected-btn {
  padding: 8px 16px;
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.delete-selected-btn:hover {
  background-color: #d32f2f;
}

.data-table-container {
  border: 1px solid lightgray;
  border-radius: 10px;
  overflow-x: auto;
  margin-left: 20px;
  margin-right: 20px;
  border-collapse: collapse;
  width: 97.25%;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  overflow-x: hidden;
}

.data-table th {
  padding: 10px 10px;
  text-align: left;
}

.data-table td {
  border-top: 1px solid lightgrey;
  border-bottom: 1px solid lightgrey;
  padding: 13px 8px;
  text-align: left;
}

.data-table input[type='text'],
.data-table input[type='email'] {
  width: 100%;
  box-sizing: border-box;
}

.data-table input[type='text'],
.data-table input[type='email'] {
  padding: 6px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.data-table td:first-child,
.data-table th:first-child {
  padding-left: 20px;
}

.data-table input[type='checkbox'] {
  transform: scale(1.25);
}

.data-table td:not(:last-child) {
  border-bottom: none;
}

.data-table td:last-child {
  border-bottom: none;
}

.data-table tr:last-child {
  margin-bottom: 0;
  border-bottom: 0;
}

.data-table th:nth-child(2),
.data-table td:nth-child(2) {
  display: none;
}

.data-table tbody tr.selected {
  background-color: gray;
  color: white;
}

.data-table tbody tr:hover {
  background-color: #f0f0f0;
}

.data-table tbody tr.selected:hover {
  background-color: lightgrey;
  color: black;
}

.icon-edit {
  padding: 4px 8px;
  background-color: transparent;
  color: black;
  border: 1px solid #000;
  border-radius: 4px;
  cursor: pointer;
}
.icon-edit:hover {
  background-color: rgb(0, 162, 255);
  color: white;
}

.icon-cancel {
  padding: 4px 10px;
  background-color: transparent;
  color: black;
  border: 1px solid #000;
  border-radius: 4px;
  cursor: pointer;
}

.icon-cancel:hover {
  background-color: #f44336;
  color: white;
}

.icon-save {
  padding: 4px 8px;
  background-color: transparent;
  color: black;
  border: 1px solid #000;
  border-radius: 4px;
  cursor: pointer;
}
.icon-save:hover {
  background-color: rgb(0, 162, 255);
  color: white;
}

.icon-delete {
  padding: 4px 8px;
  background-color: transparent;
  color: black;
  border: 1px solid #000;
  border-radius: 4px;
  cursor: pointer;
}

.icon-delete:hover {
  background-color: #f44336;
  color: white;
}

.error-msg {
  margin-left: 20px;
  color: red;
  font-weight: bold;
}

.selected-count {
  position: fixed;
  bottom: 25px;
  left: 30px;
  margin-top: 10px;
  color: gray;
}

.pagination-container {
  position: fixed;
  bottom: 20px;
  right: 40px;
  margin-top: 10px;
}

.pagination {
  margin-top: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.pagination button {
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  width: 30px;
  height: 30px;
  padding: 0;
  margin: 0 5px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.pagination button[disabled] {
  color: white;
  background-color: gray;
  cursor: not-allowed;
}
.pagination span {
  margin: 0 12px;
  font-weight: bold;
}

.pagination button:not([disabled]):hover {
  background-color: #ddd;
  color: #333;
}

@media screen and (max-width: 1238px) {
  .search-bar {
    width: 250px;
  }

  .data-table-container {
    width: 90%;
  }

  .data-table td {
    padding: 10px 10px;
  }

  .delete-selected-btn,
  .icon-edit,
  .icon-cancel,
  .icon-save,
  .icon-delete,
  .pagination button {
    font-size: 12px;
    padding: 4px 6px;
  }

  .pagination-container {
    width: 100%;
    left: 1px;
    bottom: 20px;
    font-size: 10px;
  }
  .selected-count {
    width: 30%;
    bottom: 7%;
    left: 40%;
    font-size: 10px;
    flex-wrap: wrap;
  }
}
</style>
