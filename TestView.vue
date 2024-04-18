<template>
  <!-- 관제활성화 비활성화 모달창 예시코드 -->
  <div>
    <div id="container">
      <div>
        {{ deleteCar }}
        <div>활성화 차량 리스트 ({{ activeItems.length }} )</div>
        <div class="modal-serach__group">
          <button
            class="btn diagnosis-dropdown dropdown-toggle"
            type="button"
            data-bs-toggle="dropdown"
            aria-expanded="false">
            {{ filterType }}
          </button>
          <ul class="dropdown-menu">
            <li @click="setFilter('전체')">
              <a class="dropdown-item">전체</a>
            </li>
            <li @click="setFilter('개인')">
              <a class="dropdown-item">개인</a>
            </li>
            <li @click="setFilter('법인')">
              <a class="dropdown-item">법인</a>
            </li>
          </ul>
          <div id="car-search" class="car-list__search">
            <input
              v-model="searchQuery"
              class="non-control__car"
              type="text"
              placeholder="검색어를 입력해 주세요." />

            <img
              src="@/../public/assets/images/search-icon.png"
              class="img-18x18"
              alt="search-icon" />
          </div>
        </div>
        <!-- 활성화된 아이템을 나타내는 목록 -->
        <div @click="toggleSelectAll('active')">
          <input
            type="checkbox"
            id="selectAllActive"
            v-model="selectAllActive"
            @change="selectAllItems('active')" />
          <label for="selectAllActive">전체 선택</label>
        </div>
        <button @click="removeItems">제거</button>
        <ul id="active">
          <li
            v-for="item in paginatedActiveItems"
            :key="item.id"
            @click="toggleSelection(item.id, 'active')">
            <input type="checkbox" name="active" :value="item.id" v-model="selectedItemsActive" />
            {{ item.purpose_type }} {{ item.name }} {{ item.number }}
          </li>
        </ul>
        <div class="pagination">
          <button @click="previousActivePage">이전</button>
          <span>
            <button
              v-for="pageNumber in activeTotalPages"
              :key="pageNumber"
              @click="goToActivePage(pageNumber)">
              {{ pageNumber }}
            </button>
          </span>
          <button @click="nextActivePage">다음</button>
        </div>
      </div>
      <div>
        <div>비 활성화 차량 리스트</div>
        <div class="modal-serach__group">
          <button
            class="btn diagnosis-dropdown dropdown-toggle"
            type="button"
            data-bs-toggle="dropdown"
            aria-expanded="false">
            {{ inactiveFilterType }}
          </button>
          <ul class="dropdown-menu">
            <li @click="setFilter('전체', 'inactive')">
              <a class="dropdown-item">전체</a>
            </li>
            <li @click="setFilter('개인', 'inactive')">
              <a class="dropdown-item">개인</a>
            </li>
            <li @click="setFilter('법인', 'inactive')">
              <a class="dropdown-item">법인</a>
            </li>
          </ul>
        </div>
        <input
          type="checkbox"
          id="selectAllInactive"
          v-model="selectAllInactive"
          @change="selectAllItems('inactive')" />
        <label for="selectAllInactive">전체 선택</label>
        <!-- 비활성화된 아이템을 나타내는 목록 -->
        <button @click="addItems">추가</button>
        <ul id="inactive">
          <li
            v-for="item in paginatedInactiveItems"
            :key="item.id"
            @click="toggleSelection(item.id, 'inactive')"
            :style="{
              opacity: selectedItemsInactive.includes(item.id) ? 1 : 0.4,
              transition: 'opacity 0.3s'
            }">
            <input
              type="checkbox"
              name="inactive"
              :value="item.id"
              v-model="selectedItemsInactive" />
            <div>
              {{ item.name }}
            </div>
            <div>{{ item.car_purpose }}</div>
            <div>{{ item.active }}</div>
          </li>
        </ul>
        <div class="pagination">
          <button @click="previousInactivePage">이전</button>
          <span>
            <button
              v-for="pageNumber in inactiveTotalPages"
              :key="pageNumber"
              @click="goToInactivePage(pageNumber)">
              {{ pageNumber }}
            </button>
          </span>
          <button @click="nextInactivePage">다음</button>
        </div>
      </div>
      <button @click="saveCarData">저장</button>
    </div>
  </div>
</template>

<script>
import { getCarListEnable } from '@/api/bizPayAPI'
export default {
  data() {
    return {
      filterType: '전체',
      inactiveFilterType: '전체',
      searchQuery: '',
      // 활성화된 아이템을 담는 배열
      selectedItemsActive: [],
      // 비활성화된 아이템을 담는 배열
      selectedItemsInactive: [],
      // 전체 선택/해제를 위한 변수 추가
      selectAllActive: false,
      selectAllInactive: false,
      carListEnable: [],
      carListDisable: [],
      // 활성화 목록 페이징 관련 데이터
      activeCurrentPage: 1,
      activeItemsPerPage: 5,

      // 비활성화 목록 페이징 관련 데이터
      inactiveCurrentPage: 1,
      inactiveItemsPerPage: 5,
      deleteCar: 100
    }
  },
  async mounted() {
    await this.carList()
  },

  methods: {
    // 활성화 차량 목록 필터링 타입 설정
    setFilter(type, target = 'active') {
      if (target === 'active') {
        this.filterType = type
      }
      if (target === 'inactive') {
        this.inactiveFilterType = type
      }
    },
    // 활성화 차량 필터링
    filterItem(item, target = 'active') {
      const filterType = target === 'active' ? this.filterType : this.inactiveFilterType
      if (filterType === '전체') {
        return true
      } else {
        return item.purpose_type === (filterType === '개인' ? 0 : 1)
      }
    },
    // 검색 기능
    searchItem(item) {
      return item.name.toLowerCase().includes(this.searchQuery.toLowerCase())
    },
    // 최종 비활성화 & 활성화 차량 리스트
    saveCarData() {
      console.log(this.carListEnable)
      console.log(this.carListDisable)
    },
    async carList() {
      // const response = await getCarListDisable()
      // this.carListDisable = response.data.response.body.items
      const responseTwo = await getCarListEnable()
      this.carListEnable = responseTwo.data.response.body.items
    },
    // 비활성화된 아이템을 활성화로 변경하는 메서드
    addItems() {
      const valueList = this.selectedItemsInactive.map((id) => parseInt(id))

      // 비활성화된 아이템에서 선택된 아이템을 활성화로 이동
      const itemsToActivate = this.inactiveItems.filter((item) => valueList.includes(item.id))
      this.carListDisable = this.carListDisable.filter((item) => !valueList.includes(item.id))
      this.carListEnable = [...itemsToActivate, ...this.carListEnable] // 맨 앞으로 이동

      // 선택된 아이템 초기화
      this.selectedItemsInactive = []
      // 전체 선택 체크 해제
      this.selectAllInactive = false

      // 활성화된 아이템 리스트에서 중복 제거
      this.carListEnable = [...new Set(this.carListEnable)]
    },
    // 활성화된 아이템을 비활성화로 변경하는 메서드
    removeItems() {
      const valueList = this.selectedItemsActive.map((id) => parseInt(id))

      // 선택된 활성화 차량의 수가 deleteCar를 초과하는지 확인
      if (valueList.length > this.deleteCar) {
        alert(`선택된 활성화 차량의 수가 ${this.deleteCar}대를 초과합니다.`)
        return
      }
      if (this.carListDisable.length >= this.deleteCar) {
        return alert(`비활성화 차량 리스트의 길이가 ${this.deleteCar}대를 초과할 수 없습니다.`)
      }

      // 활성화된 아이템에서 선택된 아이템을 제거하고 비활성화로 이동
      const itemsToDeactivate = this.activeItems.filter((item) => valueList.includes(item.id))
      this.carListEnable = this.carListEnable.filter((item) => !valueList.includes(item.id))

      // 비활성화 차량 리스트의 길이가 deleteCar를 초과하지 않도록 조건부 처리
      if (this.carListDisable.length + itemsToDeactivate.length <= this.deleteCar) {
        this.carListDisable = [...this.carListDisable, ...itemsToDeactivate]
      }

      // 선택된 아이템 초기화
      this.selectedItemsActive = []
      // 전체 선택 체크 해제
      this.selectAllActive = false
      console.log(this.carListEnable)
    },
    // 전체 선택/해제 기능 추가
    selectAllItems(listType) {
      if (listType === 'active') {
        this.selectedItemsActive = this.selectAllActive
          ? this.activeItems.map((item) => item.id)
          : []
      } else if (listType === 'inactive') {
        this.selectedItemsInactive = this.selectAllInactive
          ? this.inactiveItems.map((item) => item.id)
          : []
      }
    },
    // 항목 선택 토글 기능 추가
    toggleSelection(itemId, listType) {
      if (listType === 'active') {
        const index = this.selectedItemsActive.indexOf(itemId)
        if (index === -1) {
          this.selectedItemsActive.push(itemId)
        } else {
          this.selectedItemsActive.splice(index, 1)
        }
      } else if (listType === 'inactive') {
        const index = this.selectedItemsInactive.indexOf(itemId)
        if (index === -1) {
          this.selectedItemsInactive.push(itemId)
        } else {
          this.selectedItemsInactive.splice(index, 1)
        }
      }
    },
    // 전체 선택 토글 기능 추가
    toggleSelectAll(listType) {
      if (listType === 'active') {
        this.selectAllActive = !this.selectAllActive
        this.selectAllItems('active')
      } else if (listType === 'inactive') {
        this.selectAllInactive = !this.selectAllInactive
        this.selectAllItems('inactive')
      }
    },
    // 활성화 목록 페이징 관련 메소드
    previousActivePage() {
      if (this.activeCurrentPage > 1) {
        this.activeCurrentPage--
      }
    },
    nextActivePage() {
      if (this.activeCurrentPage < this.activeTotalPages) {
        this.activeCurrentPage++
      }
    },
    goToActivePage(pageNumber) {
      this.activeCurrentPage = pageNumber
    },

    // 비활성화 목록 페이징 관련 메소드
    previousInactivePage() {
      if (this.inactiveCurrentPage > 1) {
        this.inactiveCurrentPage--
      }
    },
    nextInactivePage() {
      if (this.inactiveCurrentPage < this.inactiveTotalPages[this.inactiveTotalPages.length - 1]) {
        this.inactiveCurrentPage++
      }
    },

    goToInactivePage(pageNumber) {
      this.inactiveCurrentPage = pageNumber
    }
  },
  computed: {
    // 필터링된 비활성화된 아이템을 계산하여 반환
    filteredInactiveItems() {
      return this.inactiveItems.filter(
        (item) => this.filterItem(item, 'inactive') && this.searchItem(item)
      )
    },
    // 활성화된 아이템을 계산하여 반환
    activeItems() {
      return this.carListEnable.filter((item) => this.filterItem(item, 'active'))
    },
    // 비활성화된 아이템을 계산하여 반환
    inactiveItems() {
      return this.carListDisable.filter((item) => this.filterItem(item, 'inactive'))
    },
    // 활성화 목록 페이징 관련 계산
    paginatedActiveItems() {
      const filteredItems = this.activeItems
      const startIndex = (this.activeCurrentPage - 1) * this.activeItemsPerPage
      const endIndex = startIndex + this.activeItemsPerPage
      return filteredItems.slice(startIndex, endIndex)
    },
    activeTotalPages() {
      const totalPages = Math.ceil(this.activeItems.length / this.activeItemsPerPage)
      const pageNumbers = []
      const maxPagesToShow = 3 // 최대 5개의 페이지 번호 표시

      // 현재 페이지를 중심으로 페이지 번호 계산
      let startPage = Math.max(this.activeCurrentPage - Math.floor(maxPagesToShow / 2), 1)
      let endPage = startPage + maxPagesToShow - 1
      if (endPage > totalPages) {
        endPage = totalPages
        startPage = Math.max(endPage - maxPagesToShow + 1, 1)
      }

      for (let i = startPage; i <= endPage; i++) {
        pageNumbers.push(i)
      }

      return pageNumbers
    },
    inactiveTotalPages() {
      const totalPages = Math.ceil(this.inactiveItems.length / this.inactiveItemsPerPage)
      const pageNumbers = []
      const maxPagesToShow = 3 // 최대 5개의 페이지 번호 표시

      // 현재 페이지를 중심으로 페이지 번호 계산
      let startPage = Math.max(this.inactiveCurrentPage - Math.floor(maxPagesToShow / 2), 1)
      let endPage = startPage + maxPagesToShow - 1
      if (endPage > totalPages) {
        endPage = totalPages
        startPage = Math.max(endPage - maxPagesToShow + 1, 1)
      }

      for (let i = startPage; i <= endPage; i++) {
        pageNumbers.push(i)
      }

      return pageNumbers
    },

    // 비활성화 목록 페이징 관련 계산
    paginatedInactiveItems() {
      const filteredItems = this.inactiveItems
      const startIndex = (this.inactiveCurrentPage - 1) * this.inactiveItemsPerPage
      const endIndex = startIndex + this.inactiveItemsPerPage
      return filteredItems.slice(startIndex, endIndex)
    }
  }
}
</script>

<style scoped>
#container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
}
/* 스타일 추가 - 체크박스와 레이블 간격 조절 */
input[type='checkbox'] {
  margin-right: 5px;
}
</style>
