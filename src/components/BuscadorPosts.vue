<template>
  <v-container>
    <v-card class="p-3">
      <h3 class="text-center mb-8">¿Buscas algo en concreto?</h3>
      <v-form @submit.prevent="search">
        <v-row class="justify-center">
          <v-col cols="6">
            <v-text-field
              v-model="titulo"
              filled
              outlined
              label="Título"
              required
            ></v-text-field>
          </v-col>
          <v-col cols="3">
            <v-select
              v-model="categoria"
              :items="categorias"
              label="Categoría"
              outlined
              item-value="idcategoria"
              item-text="descripcion"
              class="mb-0"
            ></v-select>
            <v-switch
              class="mt-0"
              v-model="opcionesAvanzadas"
              label="Opciones avanzadas"
              color="primary"
              value="primary"
              hide-details
            ></v-switch>
          </v-col>
          <v-col cols="3">
            <v-btn type="submit" color="green" outlined x-large>
              Buscar
            </v-btn>
          </v-col>
        </v-row>
        <v-expand-transition>
          <v-row justify="center" v-show="opcionesAvanzadas">
            <v-col cols="4">
              <v-text-field
                v-model="username"
                filled
                outlined
                label="Nombre de usuario"
                required
              ></v-text-field>
            </v-col>
            <v-col cols="4">
              <v-select
                v-model="size"
                :items="pageSizes"
                label="Número de registros por página"
                outlined
                class="mb-0"
              ></v-select>
            </v-col>
            <v-col cols="12">
              <h4 class="text-center">Criterios de ordenación</h4>
            </v-col>
            <v-col cols="3">
              <v-switch
                v-model="criteriosOrdenacion"
                color="primary"
                label="Fecha de creación"
                value="createdAt"
              ></v-switch>
            </v-col>
            <v-col cols="3">
              <v-switch
                v-model="criteriosOrdenacion"
                color="primary"
                label="Contenido"
                value="contenido"
              ></v-switch>
            </v-col>
            <v-col cols="3">
              <v-select
                v-model="tipoOrdenacion"
                :items="tiposOrdenacion"
                label="Orden"
                outlined
                item-value="id"
                item-text="desc"
                class="mb-0"
              ></v-select>
            </v-col>
          </v-row>
        </v-expand-transition>
      </v-form>
    </v-card>
  </v-container>
</template>

<script>
/* Aunque actualice el estado global del store de VUEX tiene que emitir un evento que envie un objeto con los parámetros de búsqueda para capturarlo
   en la paginación para el componente de AllPostsView
*/
const categoriaService = require("../api/categoria");
export default {
  name: "BuscadorPosts",

  data: () => ({
    titulo: "",
    categoria: "",
    categorias: [{ idcategoria: "", descripcion: "Todas" }],
    opcionesAvanzadas: false,
    username: "",
    pageSizes: [6, 9, 12, 18],
    size: 6,
    criteriosOrdenacion: [],
    tipoOrdenacion: "asc",
    tiposOrdenacion: [
      {
        id: "asc",
        desc: "Ascendente",
      },
      {
        id: "desc",
        desc: "Descendente",
      },
    ],
  }),

  computed: {
    sort() {
      return this.criteriosOrdenacion.length == 0
        ? ""
        : `${this.criteriosOrdenacion.join(",")},${this.tipoOrdenacion}`;
    },
  },

  methods: {
    search() {
      const data = {
        titulo: this.titulo,
        categoria: this.categoria,
        username: this.username,
        size: this.size,
        sort: this.sort,
        pagina: 0,
      };
      this.$store.dispatch("dispatchPosts", data);
      this.$emit("buscar-posts", data);
    },
  },

  async created() {
    const categorias = await categoriaService.findAll();
    this.categorias = [...this.categorias, ...categorias];
  },
};
</script>

<style></style>
