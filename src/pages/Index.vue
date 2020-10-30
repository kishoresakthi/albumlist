<!----------------------------------------------------------
// Created by L.kishore sakthi
// Theme : Album List and CURD operation in tabular format
------------------------------------------------------------->

<template>
  <!-- Page Layout starts -->
  <q-layout>
    <!-- header start -->
    <q-header elevated class="bg-deep-purple">
      <q-toolbar>
        <q-toolbar-title> Album App </q-toolbar-title>
      </q-toolbar>
    </q-header>
    <!-- header ends -->

    <!-- Page container starts -->
    <q-page-container>
      <div class="q-pa-md">
        <!-- Table starts -->
        <q-table
          title="Albums List"
          :data="data"
          :columns="columns"
          row-key="id"
          :selected-rows-label="getSelectedString"
          selection="multiple"
          :selected.sync="selected"
          :filter="filter"
          :loading="loading"
          :pagination.sync="pagination"
        >
          <!-- Table Search Slot-->
          <template v-slot:top-right>
            <div
              class="text-grey-8 q-gutter-md"
              style="font-size: 20px; padding-right: 20px"
            >
              <q-icon name="add_box" @click="addNew()">
                <q-tooltip> New Album </q-tooltip>
              </q-icon>
              <q-icon name="delete" @click="clickSelect(selected)">
                <q-tooltip> Bulk Delete </q-tooltip>
              </q-icon>
            </div>
            <div>
              <q-input
                dense
                debounce="300"
                v-model="filter"
                placeholder="Search"
              >
                <template v-slot:append>
                  <q-icon name="search" />
                </template>
              </q-input>
            </div>
          </template>

          <!-- Thumbnail Props -->
          <template v-slot:body-cell-thumbnailUrl="props">
            <q-td :props="props">
              <q-avatar rounded>
                <q-img :src="props.row.thumbnailUrl"> </q-img>
              </q-avatar>
            </q-td>
          </template>

          <!-- Action buttons props -->
          <template v-slot:body-cell-action="props">
            <q-td :props="props">
              <div class="text-deep-purple q-gutter-md" style="font-size: 2em">
                <q-icon
                  name="preview"
                  @click="
                    clickPreview(props.row.id, props.row.title, props.row.url)
                  "
                >
                  <q-tooltip> Preview </q-tooltip>
                </q-icon>
                <q-icon
                  name="edit"
                  @click="
                    clickEdit(props.row.id, props.row.title, props.row.url)
                  "
                >
                  <q-tooltip> Edit </q-tooltip>
                </q-icon>
                <q-icon
                  name="delete"
                  @click="clickDelete(data.indexOf(props.row))"
                >
                  <q-tooltip> Delete </q-tooltip>
                </q-icon>
              </div>
            </q-td>
          </template>
        </q-table>
        <!-- Table ends -->

        <!-- Preivew dialog -->
        <q-dialog v-model="preview">
          <q-card class="my-card">
            <q-img id="albumbanner" v-bind:src="imgsrc" />
            <q-card-section>
              <div class="row no-wrap items-center">
                <div class="col text-h6 ellipsis">
                  {{ title }}
                </div>
              </div>
            </q-card-section>
          </q-card>
        </q-dialog>
        <!-- Preview dialog ends -->

        <!-- edit dialog -->
        <q-dialog v-model="edit">
          <q-card class="my-card">
            <q-card-section class="row items-center q-pb-none">
              <div class="text-h6">Edit Album details</div>
              <q-space />
              <q-btn icon="close" flat round dense v-close-popup />
            </q-card-section>

            <q-img v-if="!tmppath" id="albumbanner" v-bind:src="imgsrc">
              <div class="absolute-bottom-right text-subtitle2">
                <input
                  ref="changeimage"
                  type="file"
                  @change="onFileChange"
                  hidden
                />
                <div @click="changeAlbum">Change image</div>
              </div>
            </q-img>

            <img v-if="tmppath" :src="tmppath" />

            <q-form @submit="updateAlbum(id)">
              <q-card-section>
                <div class="row no-wrap items-center">
                  <div class="col">
                    <q-input
                      color="deep-purple"
                      v-model="title"
                      label="Title"
                      :rules="[
                        (val) => !!val || 'Field is required',
                        (val) => val.length >= 8 || 'Minimum 8 characters',
                        (val) => val.length < 128 || 'Maximum 128 characters',
                      ]"
                    />
                  </div>
                </div>
              </q-card-section>
              <q-card-section>
                <div class="row no-wrap items-right">
                  <div class="col">
                    <q-btn color="deep-purple" type="submit" label="Update" />
                  </div>
                </div>
              </q-card-section>
            </q-form>
          </q-card>
        </q-dialog>
        <!-- edit dialog ends -->

        <!-- New album dialog -->
        <q-dialog v-model="newAlbum">
          <q-card class="my-card">
            <q-card-section class="row items-center q-pb-none">
              <div class="text-h6">New Album</div>
              <q-space />
              <q-btn icon="close" flat round dense v-close-popup />
            </q-card-section>

            <q-img v-if="!tmppath" id="albumbanner" src="../assets/dummy.jpg">
              <div class="absolute-bottom-right text-subtitle2">
                <input
                  ref="changeimage"
                  type="file"
                  @change="onFileChange"
                  hidden
                />
                <div @click="changeAlbum">Add image</div>
              </div>
            </q-img>

            <img v-if="tmppath" :src="tmppath" />

            <q-form @submit="createNew">
              <q-card-section>
                <div class="row no-wrap items-center">
                  <div class="col">
                    <q-input
                      color="deep-purple"
                      v-model="title"
                      label="Title"
                      :rules="[
                        (val) => !!val || 'Field is required',
                        (val) => val.length >= 8 || 'Minimum 8 characters',
                        (val) => val.length < 128 || 'Maximum 128 characters',
                      ]"
                    />
                  </div>
                </div>
              </q-card-section>
              <q-card-section>
                <div class="row no-wrap items-right">
                  <div class="col">
                    <q-btn color="deep-purple" type="submit" label="Create" />
                  </div>
                </div>
              </q-card-section>
            </q-form>
          </q-card>
        </q-dialog>
        <!-- New album dialog ends -->
      </div>
    </q-page-container>
    <!-- Page container starts -->
  </q-layout>
  <!-- Page Layout starts -->
</template>

<script>
export default {
  data() {
    return {
      filter: "",
      title: "",
      imgsrc: "",
      id: null,
      loading: false,
      selected: [],
      preview: false,
      edit: false,
      newAlbum: false,
      showbin: false,
      NewAlbumData: [],
      albumImage: "",
      tmppath: "",
      columns: [
        {
          name: "id",
          align: "center",
          label: "ID",
          field: "id",
          sortable: true,
        },
        {
          name: "thumbnailUrl",
          align: "center",
          label: "Album",
          field: "thumbnailUrl",
        },
        {
          name: "title",
          align: "center",
          label: "Title",
          field: "title",
          sortable: true,
        },
        {
          name: "albumId",
          align: "center",
          label: "Album ID",
          field: "albumId",
          sortable: true,
        },
        { name: "action", align: "center", label: "Action", field: "action" },
      ],
      data: [],
      pagination: {
        rowsPerPage: 7, // current rows per page being displayed
      },
    };
  },

  methods: {
    //search string
    getSelectedString() {
      return this.selected.length === 0
        ? ""
        : `${this.selected.length} record${
            this.selected.length > 1 ? "s" : ""
          } selected of ${this.data.length}`;
    },

    //Preview dialog method
    clickPreview(id, title, url) {
      this.preview = true;
      this.title = title;
      this.imgsrc = url;
      this.id = id;
    },

    //Edit dialog method
    clickEdit(id, title, url) {
      this.edit = true;
      this.title = title;
      this.imgsrc = url;
      this.id = id;
    },

    //Delete dialog method
    clickDelete(index) {
      this.$delete(this.data, index); //delete row\
      this.$q.notify({
        message: "Deleted successfully",
        color: "secondary",
        icon: "delete",
      });
    },

    //Select dialog method
    clickSelect(selected) {
      if (selected.length !== 0) {
        for (var i = 0; i < selected.length; i++) {
          this.todelete = this.data.indexOf(this.selected[i]); //delete row
          this.$delete(this.data, this.todelete);
        }
        this.selected = []; //make selected array empty again

        //success notification
        this.$q.notify({
          message: "Deleted successfully",
          color: "secondary",
          icon: "delete",
        });
      }
    },

    //Add New dialog method
    addNew() {
      this.title = "";
      this.newAlbum = true;
    },

    //Change Album input ref
    changeAlbum() {
      this.$refs.changeimage.click();
    },

    //Get file details on select
    onFileChange(event) {
      //get the file and extract its url
      this.file = event.target.files[0];
      this.tmppath = URL.createObjectURL(this.file);
      //this.tmppath = this.tmppath.replace('blob:',''); //remove blob: string form URL
      console.log(this.tmppath);
    },

    //create New album
    createNew() {
      //assign unique id for the new album entry
      this.maxid = Math.max.apply(
        Math,
        this.data.map(function (o) {
          return o.id;
        })
      );
      this.maxid = this.maxid + 1;

      //assign new id for the album
      this.albumId = Math.max.apply(
        Math,
        this.data.map(function (o) {
          return o.albumId;
        })
      );
      this.albumId = this.albumId + 1;

      //create a data_array to append
      const formdata = {
        albumId: this.albumId,
        id: this.maxid,
        title: this.title,
        url: this.tmppath,
        thumbnailUrl: this.tmppath,
      };

      // push new album data to the album list array
      if (this.data.push(formdata)) {
        //success notification
        console.log(this.data); // view pushed data in the console

        this.$q.notify({
          message: "New Album successfully",
          color: "secondary",
          icon: "done",
        });

        //clear all values and close dialog
        (this.title = ""), (this.id = ""), (this.tmppath = "");
        this.newAlbum = false;
      } else {
        this.$q.notify({
          message: "New Album successfully",
          color: "negative",
          icon: "error_outline",
        });
      }

      //push new album data to the server
      // this.$axios.post('your backend URL',
      // {
      //      "albumId": id,
      //     "id": this.maxid,
      //     "title": title,
      //     "url": this.tmppath,
      //     "thumbnailUrl": this.tmppath

      // },
      // {
      //     headers : { Authorization : "Bearer "+ this.token}
      // }
      // )
      // .then((response) => {
      //     if(response.data.status == 1)
      //     {

      //     }
      //     else if (response.data.status == 0)
      //     {
      //         this.$q.loading.hide()

      //         this.$q.notify({
      //         color: 'red-5',
      //         position: 'top',
      //         textColor: 'white',
      //         icon: 'warning',
      //         message: response.data.message
      //         })

      //     }
      // })
    },

    updateAlbum(id) {
      // find index of the specific object
      this.objIndex = this.data.findIndex((obj) => obj.id == id);
      this.data[this.objIndex].title = this.title;
      this.data[this.objIndex].url = this.tmppath;
      this.data[this.objIndex].thumbnailUrl = this.tmppath;

      //clear all values and close dialog
      (this.title = ""), (this.id = ""), (this.tmppath = "");
      this.edit = false;

      //clear all values and close dialog
      this.$q.notify({
        message: "Album Edited successfully",
        color: "secondary",
        icon: "done",
      });
    },
  },

  mounted: function () {
    this.$q.loading.show(); //show page loader

    //get album list from server
    this.$axios
      .get("https://jsonplaceholder.typicode.com/photos", {
        headers: { Authorization: "Bearer " + this.token },
      })
      .then((response) => {
        this.$q.loading.hide(); //hide page loader
        this.data = response.data;
      });
  },
};
</script>