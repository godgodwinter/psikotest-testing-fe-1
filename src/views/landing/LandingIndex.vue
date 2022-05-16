<script setup>
import axios from "axios";
import { ref } from "vue";
import Api from "@/axios/axios.js";
import Toast from "@/components/lib/Toast.js";
import ButtonOne from "@/components/atoms/ButtonOne.vue";
import ButtonTwo from "@/components/atoms/ButtonTwo.vue";
import Popper from "../../components/atoms/Popper.vue";

const file = ref(null);
const data = ref([]);
const countData = ref(0);
const diProses = ref(0);
let dataForm = {};
let dataFormDeteksi = {};

const getData = async () => {
  try {
    const response = await Api.get("testing/apiprobk/all");
    // console.log(response);
    data.value = response.data;
    countData.value = data.value.filter((item) => item.sertifikat == "belum").length;
    console.log(countData.value);
    return response;
  } catch (error) {
    Toast.danger("Warning", "Token anda kadaluarsa! Silahkan login kembali");
    console.error(error);
  }
};

const doStoreDataBackupSertifikat = async (d, index) => {
  try {
    const response = await Api.post("testing/apiprobk/api_backup", d);
    // console.log(response.data);
    Toast.success("Success", "Data Berhasil ditambahkan!");
    data.value[index].sinkron = "sudah";
    diProses.value++;
    return response.data;
  } catch (error) {
    diProses.value++;
    data.value[index].sinkron = "gagal";
    Toast.danger("Warning", "Data gagal ditambahkan!");
    console.error(error);
  }
};

const doStoreDataBackupDeteksi = async (d, index) => {
  // console.log(d);
  try {
    const response = await Api.post("testing/apiprobk/api_backup_deteksi", d);
    // console.log(response.data);
    Toast.success("Success", "Data Berhasil ditambahkan!");
    data.value[index].sinkron = "sudah";
    diProses.value++;
    return response.data;
  } catch (error) {
    diProses.value++;
    data.value[index].sinkron = "gagal";
    Toast.danger("Warning", "Data gagal ditambahkan!");
    console.error(error);
  }
};

const getDataFromApiUjianSertifikat = async (username, apiprobk_id = 0, index = 0) => {
  try {
    const response = await axios.post(
      "http://161.97.84.91:9001/api/probk/DataSertifikat_Get",
      {
        username: username,
      },
      {
        headers: {},
      }
    );
    // console.log(response);
    dataForm = response.data;
    dataForm.apiprobk_id = apiprobk_id;
    // console.log(dataForm);
    doStoreDataBackupSertifikat(dataForm, index);
    data.value[index].sertifikat = "sudah";
    return response;
  } catch (error) {
    doProsesGetApiGagal(apiprobk_id, index);
    // data.value[index].sertifikat = "gagal";
    // Toast.danger("Warning", "Proses gagal");
    console.error(error);
  }
};

const getDataFromApiUjianDeteksi = async (username, apiprobk_id = 0, index = 0) => {
  try {
    const response = await axios.post(
      "http://161.97.84.91:9001/api/probk/DataDeteksi_Get",
      {
        username: username,
      },
      {
        headers: {},
      }
    );
    // console.log(response);
    dataFormDeteksi = response.data;
    dataFormDeteksi.apiprobk_id = apiprobk_id;
    // console.log(dataFormDeteksi);
    doStoreDataBackupDeteksi(dataFormDeteksi, index);
    data.value[index].sertifikat = "sudah";
    return response;
  } catch (error) {
    doProsesGetApiGagal(apiprobk_id, index);
    // data.value[index].sertifikat = "gagal";
    // Toast.danger("Warning", "Proses gagal");
    console.error(error);
  }
};
getData();

const columns = [
  {
    label: "No",
    field: "no",
    width: "50px",
    tdClass: "text-center",
    thClass: "text-center",
  },
  {
    label: "Actions",
    field: "actions",
    sortable: false,
    width: "50px",
    tdClass: "text-center",
    thClass: "text-center",
  },
  {
    label: "Username",
    field: "username",
    type: "String",
  },
  {
    label: "GetData",
    field: "sertifikat",
    type: "String",
  },
  {
    label: "Status Get Data",
    field: "sertifikat_tgl",
    type: "String",
  },
  {
    label: "Sinkron Data",
    field: "sinkron",
    type: "String",
  },
  {
    label: "Status Sinkron Data",
    field: "sinkron_tgl",
    type: "String",
  },
];

let formData = new FormData();
const doStoreData = async (d) => {
  // console.log(data);
  try {
    // const response = await Api.post("testing/apiprobk/upload", formData);
    const response = await axios.post(
      "http://localhost:8000/api/testing/apiprobk/upload",
      formData,
      {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      }
    );
    // console.log(response.data);
    Toast.success("Success", "Data Berhasil ditambahkan!");
    getData();
    return response.data;
  } catch (error) {
    Toast.danger("Warning", "Data gagal ditambahkan!");
    console.error(error);
  }
};

const handleFileUpload = async () => {
  // formData.append("file", file.value.files[0]);
  // // debugger;
  // console.log(formData);
  // console.log("selected file", file.value.files[0]);
  // //Upload to server
  // // doStoreData();
  // // doStoreData(file.value.files);
};
const doSubmitFile = async () => {
  formData.append("file", file.value.files[0]);
  // debugger;
  // console.log(formData);
  // console.log("selected file", file.value.files[0]);
  // //Upload to server
  doStoreData();
};
// langkah-langkah
// 1.get All data belum di syncron
// doGetDataApiprobkFromBE();
// 2.async get data from api
// 3,async insert data ke be
const doSyncData = async () => {
  diProses.value = 0;
  console.log("sync data");
  // foreach then getData then push on datatable
  // 2.async insert data from api to be
  for (let i = 0; i < data.value.length; i++) {
    if (data.value[i].sertifikat == "belum") {
      getDataFromApiUjianSertifikat(data.value[i].username, data.value[i].id, i);
    }
    if (data.value[i].sertifikat == "belum") {
      getDataFromApiUjianDeteksi(data.value[i].username, data.value[i].id, i);
    }
    // console.log(data.value[i].username);
  }
};
const doProsesGetApiGagal = async (id, index) => {
  // console.log(d);
  try {
    const response = await Api.post("testing/apiprobk/gagal", {
      id: id,
    });
    // console.log(response.data);
    Toast.warning("Warning", "Update Proses gagal!");
    data.value[index].sertifikat = "gagal";
    data.value[index].sinkron = "gagal";
    diProses.value++;
    return response.data;
  } catch (error) {
    diProses.value++;
    data.value[index].sertifikat = "gagal";
    data.value[index].sinkron = "gagal";
    Toast.danger("Warning", "Data gagal diupdate!");
    console.error(error);
  }
};

const doGetDataApiprobkFromBE = async () => {
  console.log("doGetDataApiprobkFromBE");
};
const doGetDataFromApiUjian = async () => {
  // foreach then getData then push on datatable
  // 2.async get data from api
  console.log("doGetDataFromApiUjian");
};
</script>
<template>
  <!-- <div
    class="flex justify-center flex-col tracking-tight space-y-1 py-20 lg:py-14 container max-w-4xl mx-auto"
  >
    <div
      class="font-extrabold text-[40px] lg:text-[160px] text-transparent bg-clip-text bg-gradient-to-br from-sky-400 to-slate-600 text-center"
    >
      Testing
    </div>
  </div> -->
  <div class="flex gap-2 py-20 lg:py-14 container">
    <input type="file" ref="file" v-on:change="handleFileUpload()" />

    <ButtonTwo title="Upload" v-on:click="doSubmitFile()"></ButtonTwo>
    <!-- <ButtonTwo
      title="GetData Belum disinkron"
      @click="doGetDataApiprobkFromBE()"
    ></ButtonTwo>
    <ButtonTwo
      title="GetData dari Api ujian"
      @click="doGetDataFromApiUjian()"
    ></ButtonTwo> -->
    <ButtonTwo title="SinkronData" @click="doSyncData()"></ButtonTwo>
    <h1>Proses : {{ diProses }} / {{ countData * 2 }}</h1>
    <h1>Jumlah Data :{{ countData }}</h1>
  </div>

  <div class="pt-6 px-4 lg:flex flex-wrap gap-4">
    <div class="w-full lg:w-12/12">
      <div v-if="data">
        <vue-good-table
          :columns="columns"
          :rows="data"
          :search-options="{
            enabled: true,
          }"
          :pagination-options="{
            enabled: true,
            perPageDropdown: [10, 20, 50],
          }"
          styleClass="vgt-table striped bordered condensed"
          class="py-0"
        >
          <template #table-row="props">
            <span v-if="props.column.field == 'actions'">
              <div class="text-sm font-medium text-center flex justify-center">
                <ButtonEdit @click="doEditData(props.row.id)" />
                <ButtonDelete @click="doDeleteData(props.row.id)" />
                <Popper content="Reset Password" @click="doResetPassword(props.row.id)">
                  <template #content>
                    <button
                      class="text-orange-100 block rounded-sm font-bold py-1 px-1 mr-2 flex items-center hover:text-orange-300 bg-orange-400 rounded-lg"
                    >
                      <svg
                        xmlns="http://www.w3.org/2000/svg"
                        class="h-6 w-6"
                        fill="none"
                        viewBox="0 0 24 24"
                        stroke="currentColor"
                        stroke-width="2"
                      >
                        <path
                          stroke-linecap="round"
                          stroke-linejoin="round"
                          d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
                        />
                      </svg>
                    </button>
                  </template>
                </Popper>
              </div>
            </span>

            <span v-else-if="props.column.field == 'no'">
              <div class="text-center">{{ props.index + 1 }}</div>
            </span>

            <span v-else>
              {{ props.formattedRow[props.column.field] }}
            </span>
          </template>
        </vue-good-table>
      </div>
    </div>
  </div>
  <div class="container max-w-8xl mx-auto">
    <!-- Start Second Row -->
    <div class="grid grid-cols-1 md:grid-cols-2 px-4 xl:p-0 gap-4 xl:gap-6 5 my-14">
      <div class="bg-transparent p-6 rounded-xl border border-transparent pl-20 pt-10">
        <h1
          data-aos="zoom-in"
          data-aos-duration="2000"
          class="font-bold text-gray-700 text-xl sm:text-2xl md:text-2xl leading-tight mb-6"
        >
          A thoughtful way to pay
        </h1>
        <p
          data-aos="zoom-in"
          data-aos-duration="2000"
          class="text-gray-600 font-light mt-8 text-md"
        >
          Simpler App remembers your important details, so you can fill carts, not forms.
          And everything is encrypted so you can speed safely through checkout.
        </p>

        <p
          data-aos="zoom-in"
          data-aos-duration="2000"
          class="text-gray-600 font-light mt-8 text-md"
        >
          Now, you can offset the carbon emissions produced by your deliveries—for free.
          All you have to do is check out with Shop Pay, one of the first carbon-neutral
          way to pay.
        </p>
        <p
          data-aos="zoom-in"
          data-aos-duration="2000"
          class="text-gray-600 font-medium text-sm mt-4 mb-10"
        >
          Just One Way!.
        </p>

        <ButtonTwo title="More"></ButtonTwo>
      </div>

      <div
        data-aos="zoom-in"
        data-aos-duration="1000"
        class="bg-transparent p-6 rounded-xl border border-transparent flex justify-center"
      >
        <img
          data-aos="flip-left"
          data-aos-duration="2000"
          src="@/assets/img//other/img-landing2.svg"
          alt="icon"
          class="w-auto xl:w-2/3 object-cover"
        />
      </div>
    </div>
  </div>
  <!-- End Second Row -->
</template>
