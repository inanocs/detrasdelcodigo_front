<template>
  <div class="container mt-16">
    <BuscadorPosts @buscar-posts="actualizarMetadata" />

    <v-overlay :value="posts.posts.content == null">
      <v-progress-circular indeterminate size="64"></v-progress-circular>
    </v-overlay>
    <div v-if="posts.posts.content.length == 0" class="text-center">
      <h3 class="red--text">
        No se han encontrado coincidencias...
      </h3>
      <p>
        Tal vez deberías perfilar los criterios de búsqueda
      </p>
    </div>
    <div v-if="posts.posts.content.length != 0">
      <div class="row justify-center">
        <Post-Card
          v-for="post in posts.posts.content"
          :key="post.id"
          :post="post"
        />
      </div>
      <div
        class="text-center"
        v-if="posts.posts.content != null || posts.posts.content.length != 0"
      >
        <v-container>
          <v-row justify="center">
            <v-col cols="8">
              <v-container class="max-width">
                <v-pagination
                  v-model="page"
                  class="my-4"
                  :length="pageInfo.totalPages"
                  @input="cambiarPagina"
                  :total-visible="7"
                ></v-pagination>
              </v-container>
            </v-col>
          </v-row>
        </v-container>
      </div>
    </div>
  </div>
</template>

<script>
import BuscadorPosts from "@/components/BuscadorPosts.vue";
import PostCard from "@/components/PostCard.vue";
import { mapState } from "vuex";
export default {
  name: "Home",
  components: {
    PostCard,
    BuscadorPosts,
  },
  data() {
    return {
      page: 1,
      titulo: "",
      metadata: {
        titulo: "",
        pagina: 0,
        size: 6,
        sort: "",
        categoria: "",
        username: "",
      },
    };
  },
  computed: {
    ...mapState(["posts"]),
    pageInfo() {
      return {
        totalPages: this.posts.posts.totalPages,
        totalElements: this.posts.posts.totalElements,
        extraInfo: this.posts.posts.pageable,
      };
    },

    loading() {
      return this.posts.length == 0;
    },
  },

  created() {
    this.titulo = this.$route.query.titulo || "";
    this.metadata.titulo = this.titulo;
    this.$store.dispatch("dispatchPosts", this.metadata);
  },
  methods: {
    leerDatos() {
      this.titulo = this.$route.query.titulo || "";
      const actualPage = this.page - 1;
      this.$store.dispatch("dispatchPosts", this.metadata);
    },
    cambiarPagina() {
      this.leerDatos();
    },
    actualizarMetadata(data) {
      this.page = 1;
      this.metadata = data;
    },
  },
  beforeDestroy() {
    this.$store.posts = [];
  },
};
</script>

<style></style>
