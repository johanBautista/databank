<!-- eslint-disable prettier/prettier -->
<template>
  <div>
    <h1 class="text-4xl">{{ msg }}</h1>
    <div class="mt-10">
      <input type="file" @change="onChange" />
    </div>

    <div v-if="conceptoConGastos.length > 0" class="flex justify-center items-center my-20">
      <table class="min-w-75 bg-white border border-gray-200">
        <thead class="bg-blue-500">
          <tr>
            <th class="py-2 px-4 border-b">#</th>
            <th class="py-2 px-4 border-b">Concepto</th>
            <th class="py-2 px-4 border-b">Importe</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in conceptoConGastos" :key="item.concepto" :class="{ 'bg-gray-100': index % 2 === 0 }">
            <td class="py-2 px-4 border-b text-left" v-if="index > 0">
              <b>{{ index }}</b>
            </td>
            <td class="py-2 px-4 border-b text-left" v-if="index > 0">
              {{ item.concepto }}
            </td>
            <td class="py-2 px-4 border-b text-right" v-if="index > 0"
              :class="{ 'text-green-500': item.totalGastos > 0, 'text-red-500': item.totalGastos < 0 }">
              {{ item.totalGastos }} <b>€</b>
            </td>
          </tr>
        </tbody>
        <!-- Footer con el total -->
        <tfoot>
          <tr>
            <td colspan="2" class="py-2 px-4 border-t text-right font-bold">Total:</td>
            <td class="py-2 px-4 border-t text-right"
              :class="{ 'text-green-500': totalImportes > 0, 'text-red-500': totalImportes < 0 }">
              {{ totalImportes.toFixed(2) }} <b>€</b>
            </td>
          </tr>
        </tfoot>
      </table>
    </div>
  </div>
</template>

<script>
import { read, utils } from "xlsx";

export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  components: {},
  data() {
    return {
      file: [],
    };
  },
  computed: {
    conceptoConGastos() {
      const conceptos = {};
      if (!this.file) {
        return {};
      }

      // Procesar los datos para obtener la información reducida por concepto
      this.file.forEach((row) => {
        const [fecha, concepto, importe] = row;

        if (!conceptos[concepto]) {
          conceptos[concepto] = { concepto, gastos: [], totalGastos: 0 };
        }

        const importeNum = Number(importe);
        conceptos[concepto].gastos.push({ fecha, importe: importeNum });
        conceptos[concepto].totalGastos = conceptos[concepto].gastos.reduce(
          (sum, item) => sum + item.importe,
          0
        );
      });

      // Convertir el objeto de conceptos en un array
      console.log("conceptos", conceptos);
      return Object.values(conceptos);
    },
    totalImportes() {
      console.log("conceptoConGastos", this.conceptoConGastos);
      // Calcular el total de todos los importes
      return this.conceptoConGastos.reduce(
        (total, item) => total + (item.totalGastos || 0),
        0
      );
    },
  },

  methods: {
    onChange(event) {
      if (event.target.files && event.target.files.length > 0) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const wb = read(e.target.result, { type: "binary" });
          const wsname = wb.SheetNames[0];
          const ws = wb.Sheets[wsname];
          const data = utils.sheet_to_json(ws, { header: 1 });
          console.log("DATA", data);
          this.file = data;
        };

        reader.readAsBinaryString(event.target.files[0]);
      }
    },
  },
};
</script>

<style scoped></style>
