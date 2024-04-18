<template>
  <div>
    <div id="admin-check" @click="backClose($event)" :class="{ activemodal: isModalMakerOpen }">
      <div
        class="modal"
        ref="modalContent2"
        v-show="this.adminModal === 1"
        :class="{ activemodal: isModalMakerOpen }"
        :style="{
          height: this.carListResult.length > 3 && menuState === 0 ? '85vh' : 'auto'
        }">
        <div class="admin-login__btn--box" style="margin-top: 0" v-show="this.menuState === 2">
          <!-- <button id="adminLoginClose" @click="backClose($event)">취소</button> -->
          <button
            style="background: #ecf4ff; color: #2f558d"
            :style="{ opacity: isInputsValid ? 1 : 0.3 }"
            @click="nextModal">
            다음
          </button>
        </div>
        <!-- 계약 상세 정보 시작 -->
        <div v-show="this.menuState === 0">
          <div
            id="biz-pay__Remove--Date"
            class="biz-pay__car--info"
            style="margin: 30px 0px 30px 0px">
            <div id="remove--date" class="display">
              <span class="remove--car__count">계약 차량 수</span>
              <span class="remove--car__countTwo">{{ singleStateModel?.biz_car }} 대</span>
            </div>

            <div id="remove--date" class="display">
              <span class="remove--car__count">서비스 이용 기간</span>
              <span class="remove--car__countTwo"
                >{{ formatDate(singleStateModel?.service_contract_start) }} ~
                {{ formatDate(singleStateModel?.service_contract_end) }}</span
              >
            </div>

            <div id="remove--date" style="margin: 0" class="display">
              <span class="remove--car__count">계약 만료일</span>
              <span class="remove--car__countTwo warning">
                {{ formatDate(singleStateModel?.service_contract_end) }}</span
              >
            </div>
          </div>
          <span id="newREmoveSpan">비활성화 차량 </span>
          <thead id="newPayInfoThead" class="biz-table__header">
            <tr id="newPayInfoTr" class="biz-table__title" role="row border-none">
              <th>구분</th>
              <th>차량 이름</th>
              <th>차량 번호</th>
            </tr>
          </thead>
          <div id="biz-pay__tableBody" class="biz-pay__tableBody">
            <table class="table">
              <tbody class="biz-pay__table">
                <tr v-for="(item, index) in paginatedItems" :key="index">
                  <td
                    style="font-weight: 600"
                    :style="{ color: item.purpose_type === 1 ? '#00b3ff' : '#ffba60' }">
                    {{ item.purpose_type === 1 ? '법인' : '개인' }}
                  </td>
                  <td id="carName">{{ item.name }}</td>
                  <td id="carNumber">{{ item.number }}</td>
                </tr>
                <tr v-if="this.carListResult.length === 0">
                  <td
                    style="
                      text-align: center;
                      padding-left: 0px !important;
                      color: gray;
                      opacity: 0.5;
                    ">
                    비활성화 차량이 존재하지 않습니다.
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
          <div class="biz-pay__fo oter" v-if="this.carListResult.length != 0">
            <div class="pageBox">
              <div class="pagination">
                <button @click="previousPage">
                  <svg
                    width="6"
                    height="8"
                    viewBox="0 0 6 8"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg">
                    <path
                      fill-rule="evenodd"
                      clip-rule="evenodd"
                      d="M3.94375 1.42696e-08L-9.96655e-06 3.94376L0.0562258 4L-9.96521e-06 4.05623L3.94375 8L5.14038 6.80337L2.33701 4L5.14038 1.19663L3.94375 1.42696e-08Z"
                      fill="#888888" />
                  </svg>
                </button>
                <span>
                  <button
                    v-for="pageNumber in totalPages"
                    :key="pageNumber"
                    @click="goToPage(pageNumber)"
                    :class="{
                      active: activeCurrentPage === pageNumber,
                      'active-deepblue': activeCurrentPage === pageNumber,
                      'active-non': activeCurrentPage != pageNumber
                    }">
                    {{ pageNumber }}
                  </button>
                </span>
                <button @click="nextPage">
                  <svg
                    width="6"
                    height="8"
                    viewBox="0 0 6 8"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg">
                    <path
                      fill-rule="evenodd"
                      clip-rule="evenodd"
                      d="M1.33701 1.42696e-08L5.28077 3.94376L5.22454 4L5.28077 4.05623L1.33701 8L0.140381 6.80337L2.94375 4L0.140381 1.19663L1.33701 1.42696e-08Z"
                      fill="#888888" />
                  </svg>
                </button>
              </div>
            </div>
          </div>
          <div class="footer-box" style="margin: 30px 30px">
            <button @click="reapplyBiz" style="margin-right: 29px">확인</button>
          </div>
        </div>
        <!-- 결제 내역 시작 -->
        <div id="newPayHistory" v-show="this.menuState === 1">
          <thead class="biz-table__header">
            <tr id="newPayInfoTr" class="biz-table__title" role="row border-none">
              <th style="width: 180px !important; padding-left: 60px !important">결제일</th>
              <th style="width: 132px !important">계약 차량 수</th>
              <th style="width: 115px !important; padding-left: 28px !important">결제 금액</th>
              <th style="padding-left: 121px !important">결제 카드</th>
            </tr>
          </thead>
          <div id="biz-pay__tableBody" class="biz-pay__tableBody">
            <table class="table">
              <tbody class="biz-pay__table">
                <tr v-for="(item, index) in paginatedItemsTwo" :key="index">
                  <td style="padding: 0px !important; padding-left: 46px !important; width: 128px">
                    {{ formattedDate(item.contract_renewal_date) }}
                  </td>
                  <td style="padding-left: 63px !important; padding-right: 14px">
                    {{ item.biz_car }} 대
                  </td>
                  <td
                    style="
                      padding: 0px !important;
                      width: 128px !important;
                      padding-left: 28px !important;
                    ">
                    {{ formattedTotalPrice(item.total_price) }} 원
                  </td>
                  <td style="padding: 0px !important; width: 196px !important">
                    {{ item.card_name ? item.card_name.replace('[', '').replace(']', '') : '' }}
                    (****-****-****-{{ item.card_number }})
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
          <div class="biz-pay__fo oter">
            <div class="pageBox">
              <div class="pagination">
                <button @click="previousPageTwo">
                  <svg
                    width="6"
                    height="8"
                    viewBox="0 0 6 8"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg">
                    <path
                      fill-rule="evenodd"
                      clip-rule="evenodd"
                      d="M3.94375 1.42696e-08L-9.96655e-06 3.94376L0.0562258 4L-9.96521e-06 4.05623L3.94375 8L5.14038 6.80337L2.33701 4L5.14038 1.19663L3.94375 1.42696e-08Z"
                      fill="#888888" />
                  </svg>
                </button>
                <span>
                  <button
                    v-for="pageNumber in totalPagesTwo"
                    :key="pageNumber"
                    @click="goToPageTwo(pageNumber)"
                    :class="{
                      active: currentPageTwo === pageNumber,
                      'active-deepblue': currentPageTwo === pageNumber,
                      'active-non': currentPageTwo != pageNumber
                    }">
                    {{ pageNumber }}
                  </button>
                </span>
                <button @click="nextPageTwo">
                  <svg
                    width="6"
                    height="8"
                    viewBox="0 0 6 8"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg">
                    <path
                      fill-rule="evenodd"
                      clip-rule="evenodd"
                      d="M1.33701 1.42696e-08L5.28077 3.94376L5.22454 4L5.28077 4.05623L1.33701 8L0.140381 6.80337L2.94375 4L0.140381 1.19663L1.33701 1.42696e-08Z"
                      fill="#888888" />
                  </svg>
                </button>
              </div>
            </div>
          </div>
          <div class="admin-login__btn--box">
            <button style="background: #ecf4ff; color: #2f558d" @click="reapplyBiz">확인</button>
          </div>
        </div>
        <!-- 결제 내역  끝-->
      </div>
      <!-- 여기서부터 잘보면 됨 -->
      <div
        id="modalTwoState"
        style="height: unset"
        class="modal"
        ref="modalContent"
        v-show="this.adminModal === 2"
        :style="{
          width: this.adminModal === 2 ? '1000px' : 'unset',
          height: this.adminModal === 2 ? '830px' : 'unset'
        }"
        :class="{ activemodal: isModalMakerOpen }">
        <div class="admin-login__check" style="padding-bottom: 30px">
          <span @click="test">정기 결제 재신청 </span>
          <button>
            <img
              id="admin-login__check--box"
              class="img-24x24"
              src="@/../public/assets/images/close.png"
              alt="close" />
          </button>
        </div>
        <!-- 수정된 각 활성화 및 비활성화 관련 디자인 작업 -->
        <div
          style="
            display: flex;
            align-items: center;
            justify-content: space-evenly;
            width: 94%;
            padding: 0px;
            margin: 0px;
          ">
          <div class="admin-login__title--box" style="width: 373px; padding: 0">
            <div class="admin-login__title--box--titleTwo" style="padding-top: 10px">
              <span class="admin-login__title--box--title">활성화 차량 (26/8)</span>
              <transition name="fade">
                <div style="top: 218px" class="hover-div" v-if="showHover">
                  <div class="hover-main">
                    <p>
                      기존 계약보다 차량 수가 감소한 경우,<br />
                      계약 연장 이후 해당 차량 수 만큼 차량 사용이 불가능합니다.
                    </p>
                  </div>
                </div>
              </transition>
              <div class="btn-group">
                <button
                  class="btn schedule-dropdown"
                  type="button"
                  data-bs-toggle="dropdown"
                  aria-expanded="false">
                  <!-- {{ sortition }} -->
                  {{ filterType }}
                  <img
                    src="@/../public/assets/images/arrow_black.png"
                    class="img-10x6"
                    style="margin-left: 10px"
                    alt="arrow" />
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
              </div>
            </div>
            <thead style="width: unset" class="biz-table__header biz-repay__header">
              <tr class="biz-table__title" role="row border-none">
                <th style="width: 50px !important; padding: 0px !important">
                  <input
                    id="check"
                    type="checkbox"
                    v-model="selectAllActive"
                    @change="selectAllItems('active')" />
                </th>
                <th style="width: 60px !important">구분</th>
                <th style="width: 120px !important; padding: 0px !important">차량 이름</th>
                <th style="width: 143px !important; padding: 0 !important">차량 번호</th>
              </tr>
            </thead>
            <div
              id="biz-pay__tableBody biz-repay__header"
              class="biz-pay__tableBody"
              style="height: auto">
              <table class="table">
                <tbody class="biz-pay__table">
                  <tr
                    style="cursor: pointer"
                    v-for="item in paginatedActiveItems"
                    :key="item.id"
                    @click="toggleSelection(item.id, 'active')">
                    <td style="width: 50px !important; padding-left: 16px !important">
                      <input
                        name="active"
                        id="check"
                        type="checkbox"
                        v-model="selectedItemsActive"
                        :value="item.id" />
                    </td>
                    <td style="width: 60px !important; padding: 0px !important">
                      <span v-if="item.purpose_type === 0" style="color: red">개인</span>
                      <span v-if="item.purpose_type === 1" style="color: blue">법인</span>
                    </td>
                    <td style="width: 120px !important; padding: 0px !important">
                      {{ item.name }}
                    </td>
                    <td style="width: 143px !important; padding: 0 !important">
                      {{ item.number }}
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
            <div class="biz-pay__footer biz-repay__header">
              <div class="pageBox">
                <div class="pagination">
                  <button @click="previousActivePage">
                    <svg
                      width="6"
                      height="8"
                      viewBox="0 0 6 8"
                      fill="none"
                      xmlns="http://www.w3.org/2000/svg">
                      <path
                        fill-rule="evenodd"
                        clip-rule="evenodd"
                        d="M3.94375 1.42696e-08L-9.96655e-06 3.94376L0.0562258 4L-9.96521e-06 4.05623L3.94375 8L5.14038 6.80337L2.33701 4L5.14038 1.19663L3.94375 1.42696e-08Z"
                        fill="#888888" />
                    </svg>
                  </button>
                  <span>
                    <button
                      v-for="pageNumber in activeTotalPages"
                      :key="pageNumber"
                      @click="goToActivePage(pageNumber)"
                      :class="{
                        active: activeCurrentPage === pageNumber,
                        'active-deepblue': activeCurrentPage === pageNumber,
                        'active-non': activeCurrentPage != pageNumber
                      }">
                      {{ pageNumber }}
                    </button>
                  </span>
                  <button @click="nextActivePage">
                    <svg
                      width="6"
                      height="8"
                      viewBox="0 0 6 8"
                      fill="none"
                      xmlns="http://www.w3.org/2000/svg">
                      <path
                        fill-rule="evenodd"
                        clip-rule="evenodd"
                        d="M1.33701 1.42696e-08L5.28077 3.94376L5.22454 4L5.28077 4.05623L1.33701 8L0.140381 6.80337L2.94375 4L0.140381 1.19663L1.33701 1.42696e-08Z"
                        fill="#888888" />
                    </svg>
                  </button>
                </div>
              </div>
            </div>
            <!-- 수정된 각 활성화 및 비활성화 관련 디자인 작업 -->
          </div>
          <!-- 버튼 부 -->
          <div
            style="
              display: flex;
              align-items: center;
              flex-direction: column;
              justify-content: space-around;
              height: 13vh;
            ">
            <button
              style="
                background: #2f558d;
                border: none;
                border-radius: 3.91304px;
                font-weight: 500;
                font-size: 16px;
                line-height: 20px;
                letter-spacing: -0.02em;
                color: #fff;
                width: 64px;
                height: 30px;
                display: flex;
                align-items: center;
                justify-content: space-around;
              "
              @click="addItems">
              <svg
                width="6"
                height="10"
                viewBox="0 0 6 10"
                fill="none"
                xmlns="http://www.w3.org/2000/svg">
                <path
                  d="M5.12012 0.754953L0.877477 4.99759L5.12012 9.24023"
                  stroke="white"
                  stroke-width="1.5"
                  stroke-linecap="round"
                  stroke-linejoin="round" />
              </svg>
              추가
            </button>
            <button
              style="
                background: rgb(227, 64, 64);
                border: none;
                border-radius: 3.91304px;
                font-weight: 500;
                font-size: 16px;
                line-height: 20px;
                letter-spacing: -0.02em;
                color: #fff;
                width: 64px;
                height: 30px;
                display: flex;
                align-items: center;
                justify-content: space-around;
              "
              @click="removeItems">
              제외
              <svg
                width="6"
                height="10"
                viewBox="0 0 6 10"
                fill="none"
                xmlns="http://www.w3.org/2000/svg">
                <path
                  d="M0.879883 0.754953L5.12252 4.99759L0.879883 9.24023"
                  stroke="white"
                  stroke-width="1.5"
                  stroke-linecap="round"
                  stroke-linejoin="round" />
              </svg>
            </button>
          </div>
          <div class="admin-login__title--box" style="width: 373px; padding: 0">
            <div class="admin-login__title--box--titleTwo" style="padding-top: 10px">
              <span class="admin-login__title--box--title">비 활성화 차량 (26/8)</span>
              <transition name="fade">
                <div style="top: 218px" class="hover-div" v-if="showHover">
                  <div class="hover-main">
                    <p>
                      기존 계약보다 차량 수가 감소한 경우,<br />
                      계약 연장 이후 해당 차량 수 만큼 차량 사용이 불가능합니다.
                    </p>
                  </div>
                </div>
              </transition>
              <div class="btn-group">
                <button
                  class="btn schedule-dropdown"
                  type="button"
                  data-bs-toggle="dropdown"
                  aria-expanded="false">
                  {{ inactiveFilterType }}
                  <img
                    src="@/../public/assets/images/arrow_black.png"
                    class="img-10x6"
                    style="margin-left: 10px"
                    alt="arrow" />
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
            </div>
            <thead style="width: unset" class="biz-table__header biz-repay__header">
              <tr class="biz-table__title" role="row border-none">
                <th style="width: 50px !important; padding: 0px !important">
                  <input
                    id="check"
                    type="checkbox"
                    v-model="selectAllInactive"
                    @change="selectAllItems('inactive')" />
                </th>
                <th style="width: 60px !important">구분</th>
                <th style="width: 120px !important; padding: 0px !important">차량 이름</th>
                <th style="width: 143px !important; padding: 0 !important">차량 번호</th>
              </tr>
            </thead>
            <div
              id="biz-pay__tableBody biz-repay__header"
              class="biz-pay__tableBody"
              style="height: auto">
              <table class="table">
                <tbody class="biz-pay__table">
                  <tr
                    :style="{
                      opacity: selectedItemsInactive.includes(item.id) ? 1 : 0.4,
                      transition: 'opacity 0.3s'
                    }"
                    style="cursor: pointer"
                    v-for="item in paginatedInactiveItems"
                    :key="item.id"
                    @click="toggleSelection(item.id, 'inactive')">
                    <td style="width: 50px !important; padding-left: 16px !important">
                      <input
                        name="inactive"
                        id="check"
                        type="checkbox"
                        v-model="selectedItemsInactive"
                        :value="item.id" />
                    </td>
                    <td style="width: 60px !important; padding: 0px !important">
                      <span v-if="item.purpose_type === 0" style="color: red">개인</span>
                      <span v-if="item.purpose_type === 1" style="color: blue">법인</span>
                    </td>
                    <td style="width: 120px !important; padding: 0px !important">
                      {{ item.name }}
                    </td>
                    <td style="width: 143px !important; padding: 0 !important">
                      {{ item.number }}
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
            <div class="biz-pay__footer biz-repay__header">
              <div class="pageBox">
                <div class="pagination">
                  <button @click="previousInactivePage">
                    <svg
                      width="6"
                      height="8"
                      viewBox="0 0 6 8"
                      fill="none"
                      xmlns="http://www.w3.org/2000/svg">
                      <path
                        fill-rule="evenodd"
                        clip-rule="evenodd"
                        d="M3.94375 1.42696e-08L-9.96655e-06 3.94376L0.0562258 4L-9.96521e-06 4.05623L3.94375 8L5.14038 6.80337L2.33701 4L5.14038 1.19663L3.94375 1.42696e-08Z"
                        fill="#888888" />
                    </svg>
                  </button>
                  <span>
                    <button
                      v-for="pageNumber in inactiveTotalPages"
                      :key="pageNumber"
                      @click="goToInactivePage(pageNumber)"
                      :class="{
                        active: inactiveCurrentPage === pageNumber,
                        'active-deepblue': inactiveCurrentPage === pageNumber,
                        'active-non': inactiveCurrentPage != pageNumber
                      }">
                      {{ pageNumber }}
                    </button>
                  </span>
                  <button @click="nextInactivePage">
                    <svg
                      width="6"
                      height="8"
                      viewBox="0 0 6 8"
                      fill="none"
                      xmlns="http://www.w3.org/2000/svg">
                      <path
                        fill-rule="evenodd"
                        clip-rule="evenodd"
                        d="M1.33701 1.42696e-08L5.28077 3.94376L5.22454 4L5.28077 4.05623L1.33701 8L0.140381 6.80337L2.94375 4L0.140381 1.19663L1.33701 1.42696e-08Z"
                        fill="#888888" />
                    </svg>
                  </button>
                </div>
              </div>
            </div>
            <!-- 수정된 각 활성화 및 비활성화 관련 디자인 작업 -->
            <!-- 수정 안된 원본 -->
          </div>
        </div>
        <button @click="saveCarData">확인</button>
      </div>
    </div>
  </div>
</template>

<script>
import {
  adminCheck,
  getCarListEnable,
  bizPayEnable,
  getCarListDisable,
  getProgress
} from '@/api/bizPayAPI'
export default {
  name: 'BizPayRegularPaymentView',
  data() {
    return {
      // 전체 선택 및 선택
      isAllChecked: false,
      selectedItems: [],
      // login
      username: '',
      password: '',
      // input
      isInputsValid: false,
      // model
      adminModal: 2,
      // api
      carListResult: [],
      carListDisResult: [],
      payListResult: [],
      // page
      currentPage: 1,
      itemsPerPage: 10,
      showBefore: false,
      carNon: false,
      menuState: 0,
      currentPageTwo: 1,
      itemsPerPageTwo: 7,
      /////////////////// 추가
      filterType: '전체',
      inactiveFilterType: '전체',
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
  props: {
    isModalMakerOpen: Boolean,
    singleStateModel: Object
  },
  async mounted() {
    this.$nextTick(() => {
      this.$refs.modalContent2.scrollTo({
        top: 0
      })
    })
  },
  methods: {
    async test() {
      await this.carList()
    },
    /////////////////////// 추가 함수
    // 활성화 차량 목록 필터링 타입 설정
    setFilter(type, target = 'active') {
      if (target === 'active') {
        this.filterType = type
        this.activeCurrentPage = 1
      }
      if (target === 'inactive') {
        this.inactiveFilterType = type
        this.inactiveCurrentPage = 1
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
    // 항목 선택 토글 기능 추가
    toggleSelection(itemId, listType) {
      console.log('🚀 ~ toggleSelection ~ itemId, listType:', itemId, listType)
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
    async carList() {
      // const response = await getCarListDisable()
      // this.carListDisable = response.data.response.body.items
      const responseTwo = await getCarListEnable()
      this.carListEnable = responseTwo.data.response.body.items
    },
    // 최종 비활성화 & 활성화 차량 리스트
    saveCarData() {
      console.log(this.carListEnable)
      console.log(this.carListDisable)
    },
    // 활성화 목록 페이징 관련 메소드
    previousActivePage() {
      if (this.activeCurrentPage > 1) {
        this.activeCurrentPage--
      }
    },
    nextActivePage() {
      if (this.activeCurrentPage < this.activeTotalPages[this.activeTotalPages.length - 1]) {
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
    },
    /////////////////////// 추가 함수
    backReapplyBiz() {
      this.$emit('closeModal')
      this.reset()
    },
    formattedDate(dateString) {
      const [year, month, day] = dateString.split('-')
      return `${year}. ${month}. ${day}`
    },
    formatDate(dateString) {
      const date = new Date(dateString)
      const year = date.getFullYear()
      const month = date.getMonth() + 1
      const day = date.getDate()
      return `${year}년 ${month}월 ${day}일`
    },
    reset() {
      this.adminModal = 1
      this.username = ''
      this.password = ''
      this.isInputsValid = false
      this.isAllChecked = false
      this.selectedItems = []
      this.currentPageTwo = 1
      this.menuState = 0
    },
    // 모달창 관련 overflow
    updateBodyOverflow(newVal) {
      document.body.style.overflow = newVal ? 'hidden' : 'auto'
    },
    // 관리자 모달창 종료
    backClose(e) {
      let datasetId = e.target.id
      if (
        datasetId === 'admin-check' ||
        datasetId === 'admin-login__check--box' ||
        datasetId === 'adminLoginClose'
      ) {
        document.body.style.overflow = 'auto'
        this.$emit('closeModal')
        this.reset()
      }
    },
    // 스타일 활성화
    checkInput() {
      if (this.username.trim() !== '' && this.password.trim() !== '') {
        this.isInputsValid = true
      } else {
        this.isInputsValid = false
      }
    },
    // 페이징 관련 메서드
    previousPage() {
      if (this.currentPage > 1) {
        this.currentPage--
      }
      this.$nextTick(() => {
        this.$refs.modalContent.scrollTo({
          top: 0
        })
      })
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++
      }
      this.$nextTick(() => {
        this.$refs.modalContent.scrollTo({
          top: 0
        })
      })
    },
    goToPage(pageNumber) {
      this.currentPage = pageNumber
      this.$nextTick(() => {
        this.$refs.modalContent.scrollTo({
          top: 0
        })
      })
    },
    previousPageTwo() {
      if (this.currentPageTwo > 1) {
        this.currentPageTwo--
      }
    },
    nextPageTwo() {
      if (this.currentPageTwo < this.totalPages) {
        this.currentPageTwo++
      }
    },
    goToPageTwo(pageNumber) {
      this.currentPageTwo = pageNumber
    },
    async nextModal() {
      if (!this.isInputsValid) {
        return
      }
      const requestBody = {
        user_id: this.username,
        password: this.password
      }

      try {
        const response = await adminCheck(requestBody)
        if (response.data.response.header.resultCode === 401) {
          this.$store.dispatch('showToast', {
            content: '관리자가 아닙니다. \n다시 확인해주시기 바랍니다.',
            type: 'error'
          })
          return
        } else {
          this.adminModal = 2
          this.carList()
          this.$nextTick(() => {
            this.$refs.modalContent.scrollTo({
              top: 0
            })
          })
          // }
        }
      } catch (error) {
        console.log('🚀 ~ nextModal ~ error:', error)
      }
    },

    async DisCarList() {
      const response = await getCarListDisable()
      this.carListDisResult = response.data.response.body.items
      if (this.carListDisResult.length === 0) {
        this.adminModal = 2
        this.carList()
        this.$nextTick(() => {
          this.$refs.modalContent.scrollTo({
            top: 0
          })
        })
      }
    },
    async reapplyBiz() {
      let enabledCarListArray = []
      enabledCarListArray = this.carListResult.map((item) => ({ car_id: item.id }))
      const enabledCarList = {
        enabled_car_list: enabledCarListArray
      }
      try {
        const response = await bizPayEnable(this.singleStateModel.id, enabledCarList)
        console.log('🚀 ~ reapplyBiz ~ response:', response)
      } catch (error) {
        console.log('🚀 ~ reapplyBiz ~ error:', error)
      } finally {
        this.$emit('callAPI')
        this.$store.dispatch('showToast', {
          content: '재 신청이 완료 되었습니다.',
          type: 'info'
        })
        this.$emit('closeModal')
        this.reset()
      }
    },
    async callHistory(id) {
      console.log('🚀 ~ callHistory ~ id:', id.id) // 해당 아이디로서 결제 내역을 가져와야함.
      const response = await getProgress()
      this.payListResult = response.data.response.body.items
      console.log('🚀 ~ callHistory ~ respnse:', response)
    }
  },
  watch: {
    // 업데이트
    isModalMakerOpen(newVal) {
      this.updateBodyOverflow(newVal)
    },
    menuState(newValue) {
      if (newValue === 1) {
        this.callHistory(this.singleStateModel)
      }
    }
  },
  computed: {
    //////////////////////////////////// 추가
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

    //////////////////////////////////// 추가
  }
}
</script>

<style lang="scss" scoped>
@import '@/assets/bizPay/bizPayRegularPaymentView.scss';
@import '@/assets/bizPay/bizPayRemoveView.scss';
@import '@/assets/bizPay/bizPayRemoveViewV2.scss';
.admin-login__title--box {
  padding: 20px 50px;
}
.active-deepblue {
  color: $deepblue;
}

.active-non {
  color: #bbbbbb;
}
</style>
