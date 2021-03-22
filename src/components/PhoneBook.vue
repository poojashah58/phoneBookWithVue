<template>
<v-content>
    <v-container>
        <h2 align="center" class="mt-10">My Phone Book</h2>
        <v-btn
            class="mb-5 mt-10 phoneBookBtn"
            @click="openDialog()">Add New Contact</v-btn>
        <v-data-table
                :headers="headers"
                :items="contacts"
                :items-per-page="5"
                class="elevation-1"
                id="phoneBookDT"
                :sort-by.sync="sortBy"
                :sort-desc.sync="sortDesc"
            >
                <template v-slot:top>                    
                    <v-dialog  
                        v-model="dialog"
                        max-width="500px"
                        id="phoneBookDialog">
                        <v-card>
                            <v-card-title>
                                <span class="headline">{{ formTitle }}</span>
                            </v-card-title>

                            <v-card-text>
                                <v-container>
                                    <v-row>
                                        <v-col
                                            
                                        >
                                            <v-text-field
                                            v-model="editedContact.name"
                                            label="Name"
                                            :rules="[v => !!v || 'Name is required']"
                                            required
                                            ></v-text-field>
                                        </v-col>
                                        <v-col
                                            
                                        >
                                            <v-text-field
                                            id="phoneNumberTxt"
                                            v-model="editedContact.phoneNumber"
                                            label="Phone Number"
                                            @input="formatNumber"
                                            :rules="[v => !!v || 'Phone number is required',
                                              v => /^(\([0-9]{3}\)|[0-9]{3}-)[0-9]{3}-[0-9]{4}$/.test(v) || 'Phone number should be valid',]"
                                            required
                                            ></v-text-field>
                                        </v-col>                                    
                                    </v-row>
                                </v-container>
                            </v-card-text>

                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn
                                    color="gray darken-1"
                                    class="mb-5"
                                    @click="close"
                                >
                                    Cancel
                                </v-btn>
                                <v-btn
                                    color="#ffb33b"
                                    class="mb-5"
                                    :disabled="disableBtn"
                                    @click="save"
                                >
                                    Save
                                </v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                    <v-dialog v-model="deleteDialog"
                    max-width="500px">
                        <v-card class="pt-5 pb-5">
                            <div class="mb-2 pl-5">
                                Are you sure you want to delete this contact?
                            </div>
                            <v-card-actions>
                                <v-btn
                                    class="ma-1"
                                    color="grey"
                                    plain
                                    @click="closeDelete"
                                >
                                    Cancel
                                </v-btn>
                                <v-btn
                                    class="ma-1"
                                    color="error"
                                    plain
                                    @click="confirmDelete"
                                >
                                    Delete
                                </v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>                    
                </template>
                
                <template v-slot:[`item.action`]="{ item }">
                    <v-icon
                        small
                        color="#fff"
                        class="mr-2"
                        @click="editContact(item)"
                    >
                        mdi-pencil
                    </v-icon>
                    <v-icon
                        small
                        color="#fff"
                        @click="deleteContact(item)"
                    >
                        mdi-delete
                    </v-icon>
                </template>
                <template v-slot:[`item.favourite`]="{ item }">
                    <v-icon
                        small
                        class="mr-2 ml-5"
                        @click="favouriteContact(item)"
                    >
                        {{ item.favourite ? 'mdi-star' : 'mdi-star-outline' }}
                    </v-icon>                    
                </template>
            </v-data-table>
    </v-container>
</v-content>
</template>

<script>
  export default {
    data () {
      return {
        dialog:false,
        deleteDialog:false,
        isExist:-1,
        sortBy: ['favourite','name'],
        sortDesc: [true,false],
        headers: [
          {
            text: 'Name',
            align: 'start',
            value: 'name',
          },
          { text: 'Phone Number', value: 'phoneNumber'},
          { text: 'Action', value:'action', sortable:false},
          { text: 'Favourite', value:'favourite'}
        ],
        contacts: [
          {
            name: 'Tom David',
            phoneNumber: '(342)123-1234',
            favourite: true
          },
          {
            name: 'John De',
            phoneNumber: '(817)123-4567',
            favourite: true
          },
          {
            name: 'Tasha Yar',
            phoneNumber: '(410)453-5768',
            favourite: false
          }                     
        ],
        editedContact:{
            name:'',
            phoneNumber:'',
            favourite:false
        },
        defaultContact:{
            name:'',
            phoneNumber:'',
            favourite:false
        },
      }
    },
    methods:{
        openDialog(){
            this.editedContact = Object.assign({}, this.defaultContact)
            this.isExist = -1
            this.dialog = true
        },
        save(){
            if(this.validateDuplicatePhoneNumber(this.editedContact)){
                if(this.isExist > -1 ){
                    Object.assign(this.contacts[this.isExist],this.editedContact)
                }else{
                    this.contacts.push(this.editedContact)
                }
                this.close()
            }else{
                alert("Phone number already exist!");
            }            
        },
        close(){
            this.dialog = false
            this.$nextTick(() => {
                this.editedContact = Object.assign({}, this.defaultContact)
                this.isExist = -1
            })
        },
        editContact (item) {
            this.isExist = this.contacts.indexOf(item)
            this.editedContact = Object.assign({}, item)
            this.dialog = true
        },
        deleteContact(item){
            this.isExist = this.contacts.indexOf(item)
            this.editedContact = Object.assign({},item);
            this.deleteDialog=true;
        },
        closeDelete(){
            this.deleteDialog = false
            this.$nextTick(() => {
                this.editedContact = Object.assign({}, this.defaultContact)
                this.isExist = -1
            })
        },
        confirmDelete(){
            this.contacts.splice(this.isExist,1);
            this.closeDelete();
        },
        formatNumber() {
            var x = this.editedContact.phoneNumber.replace(/\D/g, '').match(/(\d{0,3})(\d{0,3})(\d{0,4})/);
            this.editedContact.phoneNumber = !x[2] ? x[1] : '(' + x[1] + ')' + x[2] + (x[3] ? '-' + x[3] : '');
        },
        validateDuplicatePhoneNumber(addedContact) {
            var allPhoneNumbers = this.contacts.map( contact => (contact.phoneNumber ))
            if(this.isExist !== -1){
                if(this.isExist == allPhoneNumbers.indexOf(addedContact.phoneNumber) || 
                allPhoneNumbers.indexOf(addedContact.phoneNumber) == -1){
                    return true;
                }
                return false;
            }else if( allPhoneNumbers.indexOf(addedContact.phoneNumber) != -1){
                return false;
            }
            return true;
        },        
        favouriteContact(item){
            if(item.favourite){
                item.favourite = false;
            }else{
                item.favourite = true;
            }
        }
    },
    computed:{
        formTitle(){
            return this.isExist === -1 ? 'Add New Contact' : 'Edit Contact'
        },
        disableBtn () {
            return this.editedContact.name === '' || this.editedContact.phoneNumber === '' || !/^(\([0-9]{3}\)|[0-9]{3}-)[0-9]{3}-[0-9]{4}$/.test(this.editedContact.phoneNumber) ? true : false;
        }
    }
  }
</script>
<style src="../assets/phoneBook.css" />
