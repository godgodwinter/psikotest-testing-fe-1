<script setup>
import { jsPDF } from "jspdf";
import { ref, onMounted, watch } from "vue";
// import html2canvas from "html2canvas";
let doc = new jsPDF("p", "pt", "A4", "potrait");
const testHtml = ref();

watch(testHtml, (newVal) => {
  doc.html(newVal, {
    callback: function (doc) {
      //   doc.save();
      //   doc.output("save", "filename.pdf"); //Try to save PDF as a file (not works on ie before 10, and some mobile devices)
      doc.output("datauristring"); //returns the data uri string
      doc.output("datauri"); //opens the data uri in current window
      doc.output("dataurlnewwindow"); //opens the data uri in new window
    },
    x: 10,
    y: 10,
  });
});
</script>
<template>
  <div ref="testHtml">
    <div class="max-w-xl">
      <h1 class="text-center">SERTIFIKAT</h1>
      <div class="px-10 py-4">
        <table class="border border-gray-400">
          <thead>
            <tr class="border border-gray-400">
              <th class="border border-gray-400">Song</th>
              <th class="border border-gray-400 text-center">Artist</th>
              <th>Year</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>The Sliding Mr. Bones (Next Stop, Pottersville)</td>
              <td>Malcolm Lockyer</td>
              <td>1961</td>
            </tr>
            <tr>
              <td>Witchy Woman</td>
              <td>The Eagles</td>
              <td>1972</td>
            </tr>
            <tr>
              <td>Shining Star</td>
              <td>Earth, Wind, and Fire</td>
              <td>1975</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="text-gray-600 text-sm px-2">
        <h1>Test heading</h1>
        <div class="card">
          <div class="card-header">Featured</div>
          <div class="card-body">
            <h5 class="card-title">Special title treatment</h5>
            <p class="card-text">
              With supporting text below as a natural lead-in to additional
              content.
            </p>
            <a href="#" class="btn btn-primary">Go somewhere</a>
          </div>
        </div>
        <div class="card">
          <div class="card-header">Featured</div>
          <div class="card-body">
            <h5 class="card-title">Special title treatment</h5>
            <p class="card-text">
              With supporting text below as a natural lead-in to additional
              content.
            </p>
            <a href="#" class="btn btn-primary">Go somewhere</a>
          </div>
        </div>
        <button class="btn btn-danger" @click="generatePdf">
          generate PDF
        </button>
      </div>
    </div>
  </div>
</template>
