<script setup>
import { computed, reactive, watchEffect } from "vue";
import { Form, Field, ErrorMessage } from "vee-validate";
import * as Yup from "yup";

const db = reactive({
  data: JSON.parse(localStorage.getItem("perpustakaan") || "[]"),
  form: {
    judulBuku: "",
    outhor: "",
    tahun: null,
    kategori: [],
    status: "",
  },
  filter: {
    kategori: "",
    status: "",
    cari: null,
  },
  selected: null,
  mode: "add",
  search: null,
});

const menampilkanData = computed(() => {
  return db.data;
});

const initData = () => {
  // (db.form = {
  //   judulBuku: null,
  //   outhor: null,
  //   tahun: null,
  //   kategori: [],
  //   status: null,
  // }),
  (db.mode = "add"), (db.selected = {});
};

const createData = (values, { resetForm }) => {
  if (db.mode === "add") {
    db.data.unshift({
      id: Math.random(),
      judulBuku: values.judulBuku,
      outhor: values.outhor,
      tahun: values.tahun,
      kategori: values.kategori,
      status: "Tersedia",
    });
    initData();
    resetForm();
  } else {
    const i = db.data.findIndex((o) => o.id === db.selected.id);
    db.data[i] = {
      ...db.selected,
      judulBuku: db.form.judulBuku,
      outhor: db.form.outhor,
      tahun: db.form.tahun,
      kategori: db.form.kategori,
      status: db.form.status,
    };
    // Reset
    initData();
    resetForm();
  }
};

const filterData = computed(() => {
  return db.data.filter((item) => {
    if (db.filter.kategori) {
      if (!item.kategori.includes(db.filter.kategori)) {
        return false;
      }
    }
    if (db.filter.status) {
      if (item.status !== db.filter.status) {
        return false;
      }
    }
    if (db.filter.cari) {
      if (!item.judulBuku.startsWith(db.filter.cari)) {
        return false;
      }
    }
    return true;
  });
});

const customKategori = (kategori) => {
  switch (kategori) {
    case "Novel":
      return " text-bg-primary ";
    default:
      return " text-bg-dark ";
  }
};

const customStatus = (status) => {
  switch (status) {
    case "Tersedia":
      return " text-bg-success ";
    case "Dipinjam":
      return " text-bg-danger ";
    default:
      return " text-bg-dark ";
  }
};

function delData(idx) {
  db.data.splice(idx, 1);
}

watchEffect(() => {
  localStorage.setItem("perpustakaan", JSON.stringify(db.data));
});

const schema = {
  kategori: (value) => {
    if (value && value.length) {
      return true;
    }

    return "You must choose a drink";
  },
};
Yup.object().shape({
  judulBuku: Yup.string()
    .required("Judul buku wajib diisi")
    .typeError("Judul buku wajib diisi"),
  tahun: Yup.string()
    .required("Tahun tidak boleh kosong")
    .typeError("Tahun harus berupa angka")
    .max(4, "tahun maksimal 4 angka")
    .matches(/^[0-9]+$/, "Tahun harus berupa angka"),
  outhor: Yup.string()
    .required("Outhor tidak boleh kosong")
    .typeError("Outhor tidak boleh kosong"),
});
</script>

<template>
  <body>
    <div class="container pt-4">
      <p class="h3 text-center mb-4 pb-3 border-bottom">
        APLIKASI PERPUSTAKAAN
      </p>
      <div class="row">
        <!-- =================================== colom inputan ==================================== -->
        <Form
          class="col-3 pe-4"
          @submit="createData"
          :validation-schema="schema"
          v-slot="{ errors }"
        >
          <!-- ======= Judul Buku ======= -->
          <div class="mb-3">
            <label for="formGroupExampleInput" class="form-label fw-bold"
              >Judul Buku</label
            >
            <Field
              name="judulBuku"
              type="text"
              class="form-control"
              id="formGroupExampleInput"
              placeholder="Example input placeholder"
              :class="{ 'is-invalid': errors.judulBuku }"
              v-model="db.form.judulBuku"
            />
            <ErrorMessage name="judulBuku" class="invalid-feedback" />
          </div>

          <!-- ======= Outhor ======= -->
          <div class="mb-3">
            <label for="formGroupExampleInput2" class="form-label fw-bold"
              >Outhor</label
            >
            <Field
              name="outhor"
              type="text"
              class="form-control"
              id="formGroupExampleInput2"
              placeholder="Another input placeholder"
              :class="{ 'is-invalid': errors.outhor }"
              v-model="db.form.outhor"
            />
            <ErrorMessage name="outhor" class="invalid-feedback" />
          </div>

          <!-- ======= Tahun ======= -->
          <div class="mb-3">
            <label for="formGroupExampleInput2" class="form-label fw-bold"
              >Tahun</label
            >
            <Field
              name="tahun"
              type="number"
              class="form-control"
              id="formGroupExampleInput2"
              placeholder="Another input placeholder"
              :class="{ 'is-invalid': errors.tahun }"
              v-model="db.form.tahun"
            />
            <ErrorMessage name="tahun" class="invalid-feedback" />
          </div>

          <!-- ======= Kategory ======= -->
          <label for="formGroupExampleInput" class="form-label fw-bold"
            >Kategory</label
          >
          <div class="row">
            <div class="col">
              <div class="form-check form-check-inline">
                <Field
                  name="kategori"
                  class="form-check-input"
                  type="checkbox"
                  id="inlineCheckbox1"
                  value="Novel"
                  v-model="db.form.kategori"
                />
                <label class="form-check-label" for="inlineCheckbox1"
                  >Novel</label
                >
              </div>
              <div class="form-check form-check-inline">
                <Field
                  name="kategori"
                  class="form-check-input"
                  type="checkbox"
                  id="inlineCheckbox2"
                  value="Majalah"
                  v-model="db.form.kategori"
                />
                <label class="form-check-label" for="inlineCheckbox2"
                  >Majalah</label
                >
              </div>
              <div class="form-check form-check-inline">
                <Field
                  name="kategori"
                  class="form-check-input"
                  type="checkbox"
                  id="inlineCheckbox3"
                  value="Kamus"
                  v-model="db.form.kategori"
                />
                <label class="form-check-label" for="inlineCheckbox3"
                  >Kamus</label
                >
              </div>
              <div class="form-check form-check-inline">
                <Field
                  name="kategori"
                  class="form-check-input"
                  type="checkbox"
                  id="inlineCheckbox4"
                  value="komik"
                  v-model="db.form.kategori"
                />
                <label class="form-check-label" for="inlineCheckbox4"
                  >Komik</label
                >
              </div>
            </div>
            <div class="col">
              <div class="form-check form-check-inline ms-3">
                <Field
                  name="kategori"
                  class="form-check-input"
                  type="checkbox"
                  id="inlineCheckbox5"
                  value="Manga"
                  v-model="db.form.kategori"
                />
                <label class="form-check-label" for="inlineCheckbox5"
                  >Manga</label
                >
              </div>
              <div class="form-check form-check-inline ms-3">
                <Field
                  name="kategori"
                  class="form-check-input"
                  type="checkbox"
                  id="inlineCheckbox6"
                  value="Ensiklopedia"
                  v-model="db.form.kategori"
                />
                <label class="form-check-label" for="inlineCheckbox6"
                  >Ensiklopedia</label
                >
              </div>
              <div class="form-check form-check-inline ms-3">
                <Field
                  name="kategori"
                  class="form-check-input"
                  type="checkbox"
                  id="inlineCheckbox7"
                  value="Biografi"
                  v-model="db.form.kategori"
                />
                <label class="form-check-label" for="inlineCheckbox7"
                  >Biografi</label
                >
              </div>
              <div class="form-check form-check-inline ms-3">
                <Field
                  name="kategori"
                  class="form-check-input"
                  type="checkbox"
                  id="inlineCheckbox8"
                  value="Naskah"
                  v-model="db.form.kategori"
                />
                <label class="form-check-label" for="inlineCheckbox8"
                  >Naskah</label
                >
              </div>
            </div>
          </div>
          <ErrorMessage name="kategori" class="invalid-feedback" />

          <!-- ======= Status ======= -->
          <div v-if="db.mode === 'edit'" class="row mb-3">
            <h6>Status</h6>
            <div class="col">
              <label class="form-check-label me-1" for="flexRadioDefault1"
                >Tersedia
              </label>
              <Field
                class="form-check-input"
                type="radio"
                name="flexRadioDefault"
                id="flexRadioDefault1"
                value="Tersedia"
                v-model="db.form.status"
              />
            </div>
            <div class="col">
              <label class="form-check-label me-1" for="flexRadioDefault1"
                >Dipinjam
              </label>
              <Field
                class="form-check-input"
                type="radio"
                name="flexRadioDefault"
                id="flexRadioDefault1"
                value="Dipinjam"
                v-model="db.form.status"
              />
            </div>
          </div>

          <!-- ======= btn Input ======= -->
          <div class="d-grid gap-2 d-md-block mt-3">
            <button class="btn btn-warning" type="submit">
              {{ db.mode === "add" ? "Simpan" : "Edit" }}
            </button>
            <button
              v-if="db.mode === 'edit'"
              @click="initForm"
              class="btn btn-danger"
            >
              Batal
            </button>
          </div>
        </Form>

        <!-- =================================== colom datalist ==================================== -->
        <div class="col-9 pt-4">
          <!-- ======= Fitering/Searcing ======== -->
          <div class="row">
            <div class="col">
              <input
                type="text"
                class="form-control"
                placeholder="Cari..."
                aria-label="First name"
                v-model="db.filter.cari"
              />
            </div>
            <div class="col">
              <select
                class="form-select"
                aria-label="Default select example"
                v-model="db.filter.kategori"
              >
                <option :value="''">Kategori</option>
                <option value="Novel">Novel</option>
                <option value="Majalah">Majalah</option>
                <option value="Kamus">Kamus</option>
                <option value="Komik">Komik</option>
                <option value="Manga">Manga</option>
                <option value="Ensiklopedia">Ensiklopedia</option>
                <option value="Biografi">Biografi</option>
                <option value="Naskah">Naskah</option>
              </select>
            </div>
            <div class="col">
              <select
                class="form-select text-dark"
                aria-label="Default select example"
                v-model="db.filter.status"
              >
                <option :value="''">Status</option>
                <option value="Tersedia">Tersedia</option>
                <option value="Dipinjam">Dipinjam</option>
              </select>
            </div>
            <div class="col">
              <button
                @click="
                  db.filter.cari = '';
                  db.filter.kategori = '';
                  db.filter.status = '';
                "
                v-if="db.filter.cari || db.filter.kategori || db.filter.status"
                class="btn btn-dark"
              >
                Reset
              </button>
            </div>
          </div>
          <!-- ======= datalist======== -->
          <table class="table table-bordered border-dark mt-3">
            <thead class="bg-warning">
              <tr>
                <th scope="col">No</th>
                <th scope="col">Judul Buku</th>
                <th scope="col">Outhor</th>
                <th scope="col">Tahun</th>
                <th scope="col">Kategori</th>
                <th scope="col">Status</th>
                <th scope="col">Aksi</th>
              </tr>
            </thead>
            <tbody v-if="filterData.length">
              <tr v-for="(item, index) in filterData" :key="item.id">
                <td>{{ index + 1 }}</td>
                <td>{{ item.judulBuku }}</td>
                <td>{{ item.outhor }}</td>
                <td>{{ item.tahun }}</td>
                <td>
                  <span
                    v-for="(kategori, index) in item.kategori"
                    :key="index"
                    :class="customKategori(index.kategori) + ' badge '"
                    >{{ kategori }}
                  </span>
                </td>
                <td :class="customStatus(item.status) + ' badge '">
                  {{ item.status }}
                </td>
                <td>
                  <button
                    @click="
                      db.mode = 'edit';
                      db.selected = item;
                      db.form = { ...item };
                    "
                    class="me-2"
                    type="button"
                  >
                    ✍
                  </button>
                  <button type="button" @click="delData(index)">❌</button>
                </td>
              </tr>
            </tbody>
            <tbody v-else>
              <tr>
                <td>
                  <span>data kosong</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </body>
</template>
<style scoped>
input,
select {
  border: 1px solid #000;
}
</style>
