<template>
  <div class="bg-gray-500">
    <h1>{{ msg }}</h1>
    <input type="file" @change="onChange" />

    <!-- 
    <div>
      <div v-for="item in conceptoConGastos" :key="item.concepto">
        <b>{{ item.concepto }} = {{ item.totalGastos }}</b>
      </div>
    </div>
     -->
    <div v-if="conceptoConGastos.length > 0">
      <table>
        <thead>
          <tr>
            <th>Concepto</th>
            <th>Importe</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in conceptoConGastos" :key="item.concepto">
            <td>{{ item.concepto }}</td>
            <td>{{ item.totalGastos }}</td>
          </tr>
        </tbody>
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

<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
