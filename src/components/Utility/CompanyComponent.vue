<template>
  <v-container mt-0>
    <v-dialog v-model="dialog" max-width="500px" scrollable @keydown.esc="dialog=false">
      <v-btn color="primary" dark slot="activator" class="mb-2">Add Company</v-btn>
      <v-card>
        <v-card-title>
          <span class="headline">{{formTitle}}</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-layout row wrap>
              <v-flex>
                <v-text-field label="Company Name" v-model="editedItem.company_Name"
                              :rules="[constants.RULES.required]"></v-text-field>
              </v-flex>
            </v-layout>

            <v-layout row wrap>
              <v-flex>
                <v-text-field label="Company Website" v-model="editedItem.company_Website"
                              :rules="[constants.RULES.required]"></v-text-field>
              </v-flex>
            </v-layout>

            <v-layout row wrap>
              <v-flex>
                <v-text-field label="Company Address" v-model="editedItem.company_Address"
                              :rules="[constants.RULES.required]"></v-text-field>
              </v-flex>
            </v-layout>

            <v-layout row wrap>
              <v-flex>
                <v-text-field label="Company Email" v-model="editedItem.company_Email"
                              :rules="[constants.RULES.required]"></v-text-field>
              </v-flex>
            </v-layout>
            <v-layout row wrap>
              <v-flex>
                <v-text-field label="Company Ceo" v-model="editedItem.company_CEO"
                              :rules="[constants.RULES.required]"></v-text-field>
              </v-flex>
            </v-layout>
            <v-layout row wrap>
              <v-flex>
                <v-text-field label="Company Founder" v-model="editedItem.company_Founder"
                              :rules="[constants.RULES.required]"></v-text-field>
              </v-flex>
            </v-layout>

            <v-layout row wrap>
              <v-flex>
                <v-text-field label="Company Information" v-model="editedItem.company_Information"
                              :rules="[constants.RULES.required]"
                              multi-line></v-text-field>
              </v-flex>
            </v-layout>

            <v-layout row wrap>
              <v-flex>
                <v-switch
                  :label="`Active: ${editedItem.company_IsActive === 1}`"
                  v-model="editedItem.company_IsActive"
                  :true-value="1"
                  :false-value="0"
                ></v-switch>
              </v-flex>
            </v-layout>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" flat @click.native="close">Cancel</v-btn>
          <v-btn color="blue darken-1" flat @click.native="save">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-layout>
      <v-flex>
        <v-text-field
          append-icon="search"
          label="Search"
          single-line
          hide-details
          v-model="search"
        ></v-text-field>
      </v-flex>
    </v-layout>

    <v-data-table
      loading="true"
      class="elevation-1"
      :items="loadedCompanies"
      :headers="headers"
      :search="search">
      <template slot="items" slot-scope="props">
        <td>{{ props.item.company_Name }}</td>
        <td>{{ props.item.company_Address }}</td>
        <td>{{ props.item.company_Website }}</td>
        <td class="text-xs-right">{{ props.item.company_IsActive===1 }}</td>
        <td class="justify-center layout px-0">
          <v-btn icon class="mx-0" @click="editItem(props.item)">
            <v-icon color="teal">edit</v-icon>
          </v-btn>
          <v-btn icon class="mx-0" @click="deleteItem(props.item)">
            <v-icon color="pink">delete</v-icon>
          </v-btn>
        </td>
      </template>
      <v-alert slot="no-results" :value="true" color="error" icon="warning">
        Your search for "{{ search }}" found no results.
      </v-alert>
    </v-data-table>
  </v-container>
</template>

<script>
  import CompanyModel from '../../models/company'
  import {EventBus} from "../../Utility/EventBus";
  import constants from '../../Utility/constants'

  export default {
    data() {
      return {
        constants:constants,
        dialog: false,
        editedIndex: -1,
        search: '',
        headers: CompanyModel.headers,
        editedItem: {
          company_Id: "",
          company_Name: "",
          company_Address: "",
          company_Email: "",
          company_CEO: "",
          company_Founder: "",
          company_Website: "",
          company_Information: "",
          company_IsActive: 0,
        },
        defaultItem: {
          company_Id: "",
          company_Name: "",
          company_Address: "",
          company_Email: "",
          company_CEO: "",
          company_Founder: "",
          company_Website: "",
          company_Information: "",
          company_IsActive: 0,
        },
      }
    },
    watch: {
      dialog(val) {
        val || this.close()
      }
    },
    computed: {
      formTitle() {
        return this.editedIndex === -1 ? 'Add Company' : 'Edit Company'
      },
      loadedCompanies() {
        return this.$store.getters.loadedCompanies
      }
    },
    methods: {
      editItem(item) {
        this.editedIndex = this.loadedCompanies.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },
      deleteItem(item) {
        const index = this.loadedCompanies.indexOf(item)
        var cnfm = confirm('Are you sure you want to delete this item?')
        if (cnfm === true) {
          this.$store.dispatch('deleteCompany', this.loadedCompanies[index])
          EventBus.$emit('notify-me', {
            data: {
              title: constants.ITEM_DELETED,
              status:constants.COLOUR_GREEN
            }
          })
        }
      },
      close() {
        this.dialog = false
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        }, 300)
      },
      save() {
        if (this.editedIndex > -1) {
          this.$store.dispatch('updateCompany', this.editedItem)
          this.items = this.$store.getters.loadedCompanies
          EventBus.$emit('notify-me', {
            data: {
              title: constants.CHANGES_SAVED,
              status:constants.COLOUR_GREEN
            }
          })
        } else {
          this.$store.dispatch('createCompany', this.editedItem)
          EventBus.$emit('notify-me', {
            data: {
              title: constants.COMPANY_CREATED,
              status:constants.COLOUR_GREEN
            }
          })
        }
        this.close()
      },
    },
    created() {

      this.items = this.$store.getters.loadedCompanies
    }
  }
</script>
