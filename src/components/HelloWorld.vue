<script setup>
import { createDirectus, rest, withToken, readItems } from '@directus/sdk';
import { ref, onMounted, computed } from 'vue';
import moment from 'moment';
import { useQuery } from "vue-query";
import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';

const itemsPerPage = ref(6);
const currentPage = ref(1);
const results = ref(null);

const client = createDirectus('https://staging-backend.teamwell.co').with(rest());

const fetchData = async () => {
  try {
    const response = await client.request(withToken('OUTeX5b3yYi4PbtOttQMbjGfZ1iG5GRK', readItems('employees')));
    results.value = response;
  } catch (error) {
    toast.error("Something went wrong", { autoClose: 1000 });
  }
};

onMounted(() => {
  const { data: result } = useQuery(["employees"],fetchData,{ retry: 3 } );
});

const dateConvert = (val, type) => {
  if (type === 'Date') {
    return moment(val).format('DD MMMM YYYY');
  } else {
    const startDate = moment(val);
    const endDate = moment();

    const yearsOfWork = endDate.diff(startDate, 'years');
    const monthsOfWork = endDate.diff(startDate, 'months') % 12;
    return `${yearsOfWork} years and ${monthsOfWork} months`
  }
};

const totalPages = computed(() => Math.ceil(results?.value?.length / itemsPerPage.value));

const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value;
  const end = start + itemsPerPage.value;
  return results?.value?.slice(start, end);
});

const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
  }
};

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
  }
};
</script>

<template>
  <div class="flex p-20 flex-col fixed top-0 left-0 bottom-0 right-0 items-center justify-center bg-gray-100 mx-auto">
    <div class="overflow-auto w-11/12">
      <div class="p-1.5 min-w-full inline-block align-middle">
        <div class="bg-white p-2 rounded-xl">
          <div class=" table border-collapse table-auto w-full divide-y divide-gray-200 dark:divide-gray-700">
            <div class="table-header-group">
              <div class="table-row">
                <div class="table-cell px-6 py-3 text-start text-[10px] font-medium text-gray-500 uppercase">name</div>
                <div class="table-cell px-6 py-3 text-start text-[10px] font-medium text-gray-500 uppercase">position
                </div>
                <div class="table-cell px-6 py-3 text-start text-[10px] font-medium text-gray-500 uppercase">responsible
                </div>
                <div class="table-cell px-6 py-3 text-start text-[10px] font-medium text-gray-500 uppercase">department
                </div>
                <div class="table-cell px-6 py-3 text-start text-[10px] font-medium text-gray-500 uppercase">debut
                  contract</div>
                <div class="table-cell px-6 py-3 text-start text-[10px] font-medium text-gray-500 uppercase">status</div>
              </div>
            </div>
            <div class="table-row-group divide-y divide-gray-200 bg-white dark:divide-gray-700 dark:bg-slate-800">
              <div class="table-row" v-for="item in paginatedData">
                <div class="table-cell px-6 py-4 whitespace-nowrap text-xs font-medium text-gray-800 dark:text-gray-200">
                  <div class="flex items-center">
                    <div class="mask mask-squircle">
                      <img width="35px" src="https://daisyui.com/images/stock/photo-1534528741775-53994a69daeb.jpg" />
                    </div>
                    <div class="ml-2">
                      <div class="text-cyan-900 text-sm">Melissa Moutchen</div>
                      <div class="text-cyan-700 font-normal text-xs">melissamoutchen@gaming1.be</div>
                      <div
                        class="badge mt-1 badge-info text-[10px] flex items-center bg-yellow-600 rounded-md bg-opacity-40">
                        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
                          class="inline-block w-[10px] h-[10px] stroke-current">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12">
                          </path>
                        </svg>
                        info
                      </div>
                    </div>
                  </div>
                </div>
                <div
                  class="table-cell px-6 py-4 whitespace-nowrap text-xs font-medium text-gray-800 dark:text-gray-200 align-middle">
                  <div class="text-cyan-900 text-sm">Digital marketer</div>
                  <div class="text-cyan-900 font-normal text-xs">Department marketing</div>
                </div>
                <div
                  class="table-cell px-6 py-4 whitespace-nowrap text-xs font-medium text-gray-800 dark:text-gray-200 align-middle">
                  <div class="text-cyan-900 text-sm">Romain Mathien</div>
                </div>
                <div
                  class="table-cell px-6 py-4 whitespace-nowrap text-xs font-medium text-gray-800 dark:text-gray-200 align-middle">
                  <div class="text-cyan-900 text-sm">Back-office</div>
                </div>
                <div
                  class="table-cell px-6 py-4 whitespace-nowrap text-xs font-medium text-gray-800 dark:text-gray-200 align-middle">
                  <div class="text-cyan-900 text-sm">{{ dateConvert(item?.date_created, 'Date') }}</div>
                  <div class="text-cyan-900 font-normal text-xs">{{ dateConvert(item?.date_created, 'Hours') }}</div>
                </div>
                <div
                  class="table-cell px-6 py-4 whitespace-nowrap text-xs font-medium text-gray-800 dark:text-gray-200 align-middle">
                  <div class="badge badge-info text-[11px] flex items-center bg-green-400 rounded-xl bg-opacity-40">
                    <span class="w-1 h-1 rounded-full bg-green-400 mr-1"></span>
                    Active
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="flex items-center justify-between border-t w-11/12 border-gray-200 px-4 py-3 sm:px-1">
      <div class="flex flex-1 justify-between sm:hidden">
        <a href="javascript:void(0)"
          class="relative inline-flex items-center rounded-md border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-700 hover:bg-gray-50">Previous</a>
        <a href="javascript:void(0)"
          class="relative ml-3 inline-flex items-center rounded-md border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-700 hover:bg-gray-50">Next</a>
      </div>
      <div class="hidden sm:flex sm:flex-1 sm:items-center sm:justify-between">
        <div>
          <p class="text-sm text-gray-700">
            Showing
            <span class="font-medium">{{ (currentPage - 1) * itemsPerPage + 1 }}</span>
            to
            <span class="font-medium">{{ currentPage == totalPages ? results.length : (currentPage - 1) * 6 + 6 }}</span>
            of
            <span class="font-medium">{{ results?.length }}</span>
            results
          </p>
        </div>
        <div>
          <!-- {{ (totalPages - 2) - (currentPage + 2)  }} -->
          <nav class="isolate inline-flex -space-x-px rounded-md shadow-sm" aria-label="Pagination">
            <a href="javascript:void(0)" @click="prevPage()"
              class="relative inline-flex items-center rounded-l-md px-2 py-2 text-gray-400 ring-1 ring-inset ring-gray-300 hover:bg-gray-50 focus:z-20 focus:outline-offset-0">
              <span class="sr-only">Previous</span>
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                stroke="currentColor" class="w-6 h-6">
                <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" />
              </svg>
            </a>
            <a @click=" currentPage = item"
              v-for="item in (totalPages - 2) - (currentPage + 2) < 1 ? Array.from({ length: Math.min(3, 3) }, (_, index) => (totalPages - 2) - (currentPage + 2) < 1 ? (totalPages - 5) + index : currentPage + index) : Array.from({ length: Math.min(3, 3) }, (_, index) => currentPage + index)"
              :key="item" href="javascript:void(0)" aria-current="page"
              :class="[item >= (totalPages - 2) ? 'hidden' : 'block', currentPage == item ? 'bg-cyan-800 text-white focus:z-20 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-cyan-800' : 'ring-1 ring-inset ring-gray-300 focus:z-20 text-gray-900 focus:outline-offset-0']"
              class="relative z-10 inline-flex items-center  px-4 py-2 text-sm font-semibold">{{ item ? item : 0 }}</a>
            <span v-if="(totalPages - 2) - (currentPage + 2) > 1"
              class="relative inline-flex items-center px-4 py-2 text-sm font-semibold text-gray-700 ring-1 ring-inset ring-gray-300 focus:outline-offset-0">...</span>

            <a @click="currentPage = item" href="javascript:void(0)" aria-current="page"
              v-for="item in Array.from({ length: totalPages }, (_, index) => index + 1).slice(-3)" :key="item"
              :class="currentPage == item ? 'bg-cyan-800 text-white focus:z-20 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-cyan-800' : 'ring-1 ring-inset ring-gray-300 focus:z-20 text-gray-900 focus:outline-offset-0'"
              class="relative z-10 inline-flex items-center  px-4 py-2 text-sm font-semibold">{{ item ? item : 0 }}</a>
            <a href="javascript:void(0)" @click="nextPage()"
              class="relative inline-flex items-center rounded-r-md px-2 py-2 text-gray-400 ring-1 ring-inset ring-gray-300 hover:bg-gray-50 focus:z-20 focus:outline-offset-0">
              <span class="sr-only">Next</span>
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                stroke="currentColor" class="w-6 h-6">
                <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
              </svg>
            </a>
          </nav>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
