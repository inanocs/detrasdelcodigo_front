<template>
  <v-container class="mt-15" :fluid="true">
    <div class="row ">
      <div class="col-md ">
        <div class="bg-white shadow rounded overflow-hidden">
          <div class="px-4 pt-0 pb-4 cover">
            <div class="media align-items-end profile-head">
              <div class="profile mr-3">
                <img
                  :src="user.avatar"
                  alt="..."
                  width="130"
                  class="rounded mb-2 img-thumbnail username-profile--avatar"
                /><button
                  v-if="puedeEditar"
                  class="btn btn-danger white--text btn-sm btn-block "
                  @click.prevent="displayModal('editarperfil')"
                >
                  Editar perfil
                </button>
              </div>
              <div class="media-body mb-5 text-white">
                <h4 class="mt-0 mb-0">
                  {{ user.nombre }} {{ user.apellidos }}
                  <v-chip color="primary" small>{{
                    user.rol.descripcion | capitalize
                  }}</v-chip>
                </h4>
                <p class="small mb-4">
                  <v-icon color="white">mdi-account</v-icon>@{{ user.username }}
                </p>
              </div>
            </div>
          </div>
          <div class="bg-light p-4 d-flex justify-content-end text-center">
            <ul class="list-inline mb-0">
              <li class="list-inline-item">
                <h5 class="font-weight-bold mb-0 d-block">
                  {{ postsCreados }}
                </h5>
                <small class="text-muted">
                  <i class="fas fa-image mr-1"></i>Posts creados</small
                >
              </li>
              <li class="list-inline-item">
                <h5 class="font-weight-bold mb-0 d-block">
                  {{ totalComentarios }}
                </h5>
                <small class="text-muted">
                  <i class="fas fa-user mr-1"></i>Comentarios recibidos</small
                >
              </li>
            </ul>
          </div>
          <div class="px-4 py-3">
            <h5 class="mb-0">About</h5>
            <div class="p-4 rounded shadow-sm bg-light">
              <p class="font-italic mb-0">
                Lorem ipsum dolor sit amet, consectetur adipisicing elit. Maxime
                illo maiores ratione facere, magni sit earum iure vero fugit
                quos nihil repudiandae facilis animi expedita reiciendis
                deleniti a alias distinctio!
              </p>
            </div>
          </div>
          <div class="py-4 px-4">
            <div class="d-flex align-items-center justify-content-between mb-3">
              <h5 class="mb-0">Últimos posts</h5>
              <v-btn
                v-if="puedeEditar && userposts.length !== 0"
                color="green"
                outlined
                class="ml-5"
                :to="{ path: '/crearpost' }"
                >Crea un nuevo post</v-btn
              >
            </div>
            <div
              class="row"
              v-if="
                userposts == null || !userposts.length || userposts.length == 0
              "
            >
              <div class="col-12 warning--text text-center">
                <v-icon color="warning" class="mr-1">mdi-alert</v-icon>
                <span>No hay posts</span>
                <v-btn
                  v-if="puedeEditar"
                  color="green"
                  outlined
                  class="ml-5"
                  :to="{ path: '/crearpost' }"
                  >Crea un nuevo post</v-btn
                >
              </div>
            </div>
            <div
              class="row justify-lg-start mx-auto"
              v-if="
                userposts != null || userposts.length || userposts.length != 0
              "
            >
              <v-hover
                v-slot="{ hover }"
                v-for="post in userposts"
                :key="post.id"
              >
                <div
                  class="col-lg-6 mb-2 mr-2 ml-9 p-0 mt-3 posts-box mr-lg-16 ml-lg-16 mb-lg-5"
                >
                  <v-expand-transition>
                    <div
                      v-if="hover"
                      class="d-flex flex-column text-center transition-fast-in-fast-out black darken-1 v-card--reveal display-1 white--text"
                      style="height: 100%;"
                    >
                      <h2>{{ post.titulo }}</h2>
                      <p class="hover--paragraph mt-0">
                        {{ post.descripcion | descripcionPost }}
                      </p>
                      <div
                        class="d-flex align-items-center justify-content-between"
                      >
                        <v-btn
                          small
                          color="primary"
                          rounded
                          class="white--text"
                          >{{ post.categoria.descripcion }}</v-btn
                        >
                        <div class="ml-5 d-flex align-items-center">
                          <v-icon small color="white">mdi-comment</v-icon>
                          <span class="ml-2 hover--span">{{
                            post.comentarios.length
                          }}</span>
                        </div>
                      </div>

                      <v-btn
                        :to="{ path: `/posts/${post.idpost}` }"
                        class="mt-4 text-center white--text"
                        color="green"
                        filled
                        >Ir al post</v-btn
                      >
                    </div>
                  </v-expand-transition>
                  <img
                    :src="post.portada"
                    alt=""
                    class="posts-box__img rounded shadow-sm"
                  />
                </div>
              </v-hover>
            </div>
          </div>
          <div
            class="text-center"
            v-if="userposts != null && pageInfo.totalPages > 1"
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
    </div>
    <ModalComponent
      :visualization="modalVisualization"
      :activated="modalActivado"
      @closeModal="resetModalValues"
    />
  </v-container>
</template>

<script>
import { mapState } from "vuex";
import ModalComponent from "@/components/ModalComponents.vue";
const postService = require("../api/posts");
const authService = require("../api/auth");
const jwtUtil = require("../util/jwt");
export default {
  name: "UsernameView",
  components: { ModalComponent },
  computed: {
    ...mapState({
      user: (state) => state.users.user,
      userposts: (state) => state.users.userposts.content,
      pageInfo: (state) => ({
        totalPages: state.users.userposts.totalPages,
        totalElements: state.users.userposts.totalElements,
        size: state.users.userposts.size,
        extraInfo: state.users.userposts.pageable,
      }),
    }),

    totalComentarios() {
      return this.extradata == null ? 0 : this.extradata.totalPostComments;
    },

    postsCreados() {
      return this.extradata == null ? 0 : this.extradata.totalPostsCreated;
    },
  },
  data() {
    return {
      usernameParam: this.$route.params.username,
      hover: false,
      extradata: null,
      page: 1,
      puedeEditar: false,
      modalVisualization: "",
      modalActivado: false,
    };
  },

  async created() {
    this.$store.dispatch("dispatchUser", this.usernameParam);
    this.$store.dispatch("dispatchUserPosts", {
      username: this.usernameParam,
      titulo: "",
      pagina: 0,
      size: 4,
      sort: "createdAt,desc",
      tag: "",
      categoria: "",
    });

    this.extradata = await postService.getPostsExtraData(this.usernameParam);

    if (authService.getUserSessionToken()) {
      const decodedtoken = jwtUtil.decodeToken(
        authService.getUserSessionToken()
      );
      if (decodedtoken.username === this.usernameParam) {
        this.puedeEditar = true;
      }
    }
  },
  methods: {
    leerDatos() {
      const actualPage = this.page - 1;
      this.$store.dispatch("dispatchPosts", {
        username: this.usernameParam,
        titulo: "",
        pagina: actualPage,
        size: 4,
        sort: "createdAt,desc",
        tag: "",
        categoria: "",
      });
    },
    cambiarPagina() {
      this.leerDatos();
    },
    displayModal(ruta) {
      this.modalVisualization = ruta;
      this.modalActivado = true;
    },
    resetModalValues(payload) {
      this.modalVisualization = "";
      this.modalActivado = payload;
    },
  },
  filters: {
    descripcionPost(descripcion) {
      return descripcion.length > 35
        ? `${descripcion.substring(0, 35)}...`
        : descripcion;
    },
    capitalize: function(value) {
      if (!value) return "";
      value = value.toString();
      return value.charAt(0).toUpperCase() + value.slice(1);
    },
  },
};
</script>

<style>
.posts-box {
  overflow: hidden;
  max-width: 387px;
  height: 232px;
}

.posts-box__img {
  object-position: center;
  object-fit: cover;
  height: 232px;
  max-width: 100%;
}

.profile-head {
  transform: translateY(3rem);
}

.cover {
  background-image: url(https://images.unsplash.com/photo-1530305408560-82d13781b33a?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1352&q=80);
  background-size: cover;
  background-repeat: no-repeat;
}

body {
  background: #654ea3;
  background: linear-gradient(to right, #e96443, #904e95);
  min-height: 100vh;
}
.v-card--reveal {
  align-items: center;
  font-size: 12px;
  bottom: 0;
  justify-content: center;
  opacity: 0.8;
  position: absolute;
  width: 100%;
}

.hover--paragraph {
  font-size: 1.3rem;
}
.hover--span {
  font-size: 1.1rem;
}

.username-profile--avatar {
  height: 130px;
  width: 130px;
  object-fit: cover;
}
</style>
