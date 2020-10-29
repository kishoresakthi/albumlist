<!----------------------------------------------------------
// Created by L.kishore sakthi
// Theme : Album List and CURD operation in tabular format
------------------------------------------------------------->

<template>
<!-- Page Layout starts -->
  <q-layout>
    <!-- header start -->
    <q-header 
    elevated
    class="bg-deep-purple">
      <q-toolbar>
        <q-toolbar-title>
          Album App
        </q-toolbar-title>
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
          <div class="text-grey-8 q-gutter-md" style="font-size: 20px; padding-right : 20px">
            <q-icon name="add_box" @click="addNew()">
              <q-tooltip>
                New Album
              </q-tooltip>
            </q-icon>
            <q-icon name="delete" @click="clickSelect(selected)">
              <q-tooltip>
                Bulk Delete
              </q-tooltip>
            </q-icon>
          </div> 
          <div>
            <q-input dense debounce="300" v-model="filter" placeholder="Search">
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
                  <q-img
                    :src="props.row.thumbnailUrl"
                    >
                </q-img>
              </q-avatar>
            </q-td>
      </template>

      <!-- Action buttons props -->
      <template v-slot:body-cell-action="props">
            <q-td :props="props">
              <div class="text-deep-purple q-gutter-md" style="font-size: 2em">
                <q-icon name="preview" @click="clickPreview(props.row.id, props.row.title, props.row.url)">
                  <q-tooltip>
                    Preview
                  </q-tooltip>
                </q-icon>
                <q-icon name="edit" @click="clickEdit(props.row.id, props.row.title, props.row.url)">
                  <q-tooltip>
                    Edit
                  </q-tooltip>
                </q-icon>
                <q-icon name="delete" @click="clickDelete(data.indexOf(props.row))">
                  <q-tooltip>
                    Delete
                  </q-tooltip>
                </q-icon>
              </div>
            </q-td>
      </template>

      </q-table>
      <!-- Table ends -->

      <!-- Preivew dialog -->
      <q-dialog v-model="preview">
        <q-card class="my-card">
          <q-img id="albumbanner" v-bind:src = "imgsrc" />
          <q-card-section>
            <div class="row no-wrap items-center">
              <div class="col text-h6 ellipsis">
                {{title}}
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

          <q-img id="albumbanner" v-bind:src = "imgsrc">
              <div class="absolute-bottom-right text-subtitle2">
                <input ref="changeimage" type="file" hidden/>
                  <div @click="changeAlbum">Change image </div>
              </div>
          </q-img> 

          <q-card-section>
            <div class="row no-wrap items-center">
              <div class="col">
                <q-input color="deep-purple" v-model="title" label="Title" value="title" />
              </div>
            </div>
          </q-card-section>
          <q-card-section>
            <div class="row no-wrap items-right">
              <div class="col">
                <q-btn color="deep-purple" label="Save" />
              </div>
            </div>
          </q-card-section>
          
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

          <q-img id="albumbanner" src="../assets/dummy.jpg">
              <div class="absolute-bottom-right text-subtitle2">
                <input ref="changeimage" type="file" hidden/>
                  <div @click="changeAlbum">Change image </div>
              </div>
          </q-img> 

          <q-card-section>
            <div class="row no-wrap items-center">
              <div class="col">
                <q-input color="deep-purple" v-model="title" label="Title" />
              </div>
            </div>
          </q-card-section>
          <q-card-section>
            <div class="row no-wrap items-right">
              <div class="col">
                <q-btn color="deep-purple" label="Save" />
              </div>
            </div>
          </q-card-section>
          
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

  data () {
    return {
      filter: '',
      title: '',
      imgsrc: '',
      id: '',
      loading: false,
      selected: [],
      preview : false,
      edit : false,
      newAlbum : false,
      showbin: false,
      albumImage: '',
      columns: [
        { name: 'thumbnailUrl', align: 'center', label: 'Album', field: 'thumbnailUrl'},
        { name: 'title', align: 'center', label: 'Title', field: 'title', sortable: true},
        { name: 'id', align: 'center', label: 'ID', field: 'id', sortable: true},
        { name: 'action', align: 'center', label: 'Action', field: 'action'},
      ],
      data: [],
      pagination: {
        rowsPerPage: 7 // current rows per page being displayed
      },
    }
  },
  
  methods: {
    //search string
    getSelectedString () {
      return this.selected.length === 0 ? '' : `${this.selected.length} record${this.selected.length > 1 ? 's' : ''} selected of ${this.data.length}`
    },

    //Preview dialog method
    clickPreview (id, title, url) {
      this.preview = true
      this.title = title
      this.imgsrc = url
      this.id = id
    },

    //Edit dialog method
    clickEdit (id, title, url) {
      this.edit = true
      this.title = title
      this.imgsrc = url
      this.id = id
    },

    //Delete dialog method
    clickDelete (index) {
       this.$delete(this.data, index) //delete row\
       this.$q.notify({
        message: 'Deleted successfully',
        color: 'secondary',
        icon: 'delete'
      })
    },

    //Select dialog method
     clickSelect (selected) {

      if(selected.length !== 0){
        for(var i = 0; i < selected.length; i++) {
          this.todelete = this.data.indexOf(this.selected[i]) //delete row
          this.$delete(this.data, this.todelete)
        }
        this.selected = [] //make selected array empty again

        //success notification
        this.$q.notify({
          message: 'Deleted successfully',
          color: 'secondary',
          icon: 'delete'
        })
      } 
    },

    //Add New dialog method
    addNew (){
      this.title = ''
      this.newAlbum = true
    },

    //Change Album dialog method
    changeAlbum (){
      this.$refs.changeimage.click()
    }
  },

  mounted : function () {
        this.$q.loading.show() //show page loader

        //get album list from server
        this.$axios.get('https://jsonplaceholder.typicode.com/photos', 
          { 
            headers : { Authorization : "Bearer "+ this.token}
          }
        )
        .then((response) => {

          this.$q.loading.hide() //hide page loader
          this.data = response.data
        })
    
  },
}
</script>