
<template>
  <div>
    <v-layout
      justify-center
      align-center
    >
      <v-responsive
      class="bgright2 white--text"
      height="130"
      > 
       <v-card height="100%" style="background-color:#ffffffcf;">
          <v-container>
          <v-layout row fill-height  >
            <div>
              <div class="display-1">Dashboard</div>
              <div class="body-1 font-weight-light ">"It is quality rather than quantity that matters." - Lucius Annaeus Seneca</div>
            </div>
          </v-layout>
          </v-container>
        </v-card>
      </v-responsive>
    </v-layout>
    <v-container grid-list-xs>
    <v-layout wrap>
      <v-flex
         xs12
         class="mb-3"
      >
         <v-sheet height="500">
         <v-calendar
            color="teal"
            ref="calendar"
            v-model="start"
            :type="type"
            :end="end"
         >

          <template
            slot="day"
            slot-scope="{ date }"
           >
            <template v-for="event in eventsMap[date]">
              <v-menu
                :key="event.title"
                v-model="event.open"
                full-width
                offset-x
              >
                <div
                  v-if="!event.time"
                  slot="activator"
                  v-ripple
                  class="my-event"
                  v-html="event.title"
                ></div>
                <v-card
                  color="grey lighten-4"
                  min-width="350px"
                  flat
                >
                 
                  <v-card-title primary-title>
                    <span v-html="event.details"></span>
                  </v-card-title>
                  <v-card-actions>
                    <v-btn
                      flat
                      color="secondary"
                    >
                      Cancel
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-menu>
            </template>
          </template>
         
         </v-calendar>
         </v-sheet>
      </v-flex>

      <v-flex
      sm4
      xs12
      class="text-sm-left text-xs-center"
    >
      <v-btn @click="$refs.calendar.prev()">
        <v-icon
          dark
          left
        >
          keyboard_arrow_left
        </v-icon>
        Prev
      </v-btn>
    </v-flex>
    <v-flex
      sm4
      xs12
      class="text-xs-center"
    >
      <v-select
        v-model="type"
        :items="typeOptions"
        label="Type"
      ></v-select>
    </v-flex>
    <v-flex
      sm4
      xs12
      class="text-sm-right text-xs-center"
    >
      <v-btn @click="$refs.calendar.next()">
        Next
        <v-icon
          right
          dark
        >
          keyboard_arrow_right
        </v-icon>
      </v-btn>
    </v-flex>
     
   </v-layout>
   </v-container>

    <v-container  >
      
      
    <v-toolbar 
      class="teal lighten-3" flat 
      
    >
      <v-toolbar-title class="teal--text">Activity of Logs</v-toolbar-title>
      <!-- <v-divider
        class="mx-2"
        inset
        vertical
      ></v-divider> -->
      <v-spacer></v-spacer>
      <v-text-field
        v-model="search"
        append-icon="search"
        label="Search"
        class="jiebtn"
        solo-inverted
        single-line
        dark
        color="teal"
        hide-details
        flat
      ></v-text-field>
    </v-toolbar>
    <v-data-table
      :headers="headers"
      :items="getSales"
      class="mt-3 elevation-1 teal"
      :loading="categoryLoadingdata"
      :search="search"
    >
     
      <template v-if="props.item.keyIndex != 'test'" slot="items" slot-scope="props">
        
        <td>{{ isToday(props.item.date) }}</td>
        <td>{{ props.item.name }}</td>
        <td>{{ isCashier(props.item.incharge) }}</td>
        <td> <v-chip label class="teal lighten-2 white--text" v-for="item in props.item.test" :key="item.key">Serial#: {{ item.serial }} || {{ item.quantity }}</v-chip></td>
        <td class="success--text"> (₱) {{props.item.total}} </td>
        
      </template>
      <div slot="no-results" :value="true">
        <v-layout align-center="">
         <v-icon color="error">mdi-alert</v-icon>
         <span class="error--text ml-2">Your search for "{{ search }}" found no results.</span>
        </v-layout>
      </div>
    </v-data-table>
    
    <v-dialog
      persistent
      :value="dialogEdit"
      width="500"
    >
      <v-card>
          <v-card-title>
            <span class="headline"> {{headlineTitle}}  </span>
          </v-card-title>
          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12  >
                  <v-text-field :error-messages="cateValidmessage" color="teal" ref="cnameRefs" v-model="editedItem.cname" label="Category name"></v-text-field>
                </v-flex>
                <v-flex xs12  >
                  <v-text-field color="teal"  v-model="editedItem.description" label="Description"></v-text-field>
                </v-flex>
              </v-layout>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="teal darken-1" flat @click="cancel">Cancel</v-btn>
            <v-btn v-if="headlineTitle != 'Add category'" :disabled="saveValid" color="teal darken-1" flat @click="editItemSave(editedItem)">Save</v-btn>
            <v-btn v-else color="teal darken-1"  :disabled="saveValid"  flat @click="addCategoryData">Save</v-btn>
          </v-card-actions>
        </v-card>
    </v-dialog>

    <v-dialog
      v-model="deletedialog"
      width="300"
    >
      <v-card>
        <v-card-title>
          <span class="body-2 mb-2"> Are you sure you want to delete this item?  </span>
          <span class="caption error--text"> All products that included in this category will also be deleted</span>

        </v-card-title>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            flat
            @click="deletedialog = false"
          >
            Cancel
          </v-btn>
          <v-btn
            color="error"
            flat
            @click="deleteItembtn"
          >
            Delete
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    </v-container>

    <v-snackbar
      bottom
      right
      color="white"
      v-model="snackbar"
    > 
    <span class="teal--text">{{snackbarMessage}}</span>
    </v-snackbar>
  </div>
</template>

<script>
  import moment from 'moment'
  import firebase from 'firebase'
  export default {
    data () {
      return {
         events: [
            {
               title: 'Vacation',
               details: 'Going to the beach!',
               date:  '2019-02-28',
               open: false
            },
         ],


         type: 'month',
         start: '2019-02-01',
         end: '2019-02-01',

         typeOptions: [
         // { text: 'Day', value: 'day' },
         // { text: '4 Day', value: '4day' },
         // { text: 'Week', value: 'week' },
         { text: 'Month', value: 'month' },
         // { text: 'Custom Daily', value: 'custom-daily' },
         // { text: 'Custom Weekly', value: 'custom-weekly' }
         ],

        search: '',
        headers: [
          { text: 'Date', value: 'date' , sortable: true},
          { text: 'Name', value: 'name' , sortable: true},
          { text: 'Cashier', value: 'incharge' , sortable: true},
          { text: 'Items', value: 'items', sortable: false},
          { text: 'Total Amount', value: 'cash', sortable: false },
        ],
        categorys: [],
        categoryLength: '',
        categoryLoadingdata: false,
        dialogEdit: false,
        editedItem: {
          cname: '',
          dummycname: '',
          description:'',
          keyIndex:''
        },
        headlineTitle: '',
        snackbar: false,
        snackbarMessage: '',
        deletedialog: false,
        deleteItem: {
          category: '',
          key:''
        },
      }
    },
    created() {
      this.getCategoryData()
    },
    computed: {
      calendarSales() {
         let events = []
         _.forEach(this.getSales, function(value, key){
            var date = moment(value.date).format('YYYY-MM-DD');
            var total = value.cash
            var get = _.filter(events,['date', date])
            console.log(get[0])
            if(get.length != 0) {
               // total += get[0].details * 1
               events.splice(events.indexOf(get[0]), 1)
               events.push({
               // (₱) ' + 
               //
               // title:  _.add(value.cash,get[0].details),
               title:  1 * value.cash + get[0].details * 1,
               date: date,
               details:  + 1 * value.cash + get[0].details * 1,
               open: false,
               keyIndex: value.keyIndex
            })
            } else {
               events.push({
                  title:  value.cash,
                  date: date,
                  details: total,
                  open: false,
                  keyIndex: value.cash
               })
            }
         })
         
            // console.log(events)
         return events
      },
      eventsMap () {
        const map = {}
        this.calendarSales.forEach(e => (map[e.date] = map[e.date] || []).push(e))
        return map
      },
      getSales() {
         console.log()
         let filter = _.filter(this.$store.getters.sales, 'incharge')
         return filter
      },
      cateValidmessage() {
        var droptest = _.dropRight(this.categoryData)
        var findThis = _.find(droptest, ['cname', _.capitalize(this.editedItem.cname)])
        if(findThis && _.capitalize(this.editedItem.cname) !=  _.capitalize(this.editedItem.dummycname) ) {
          return 'Category name already exist'
        } else {
          return ''
        }
      },
      cateValid() {
        var droptest = _.dropRight(this.categoryData)
        var findThis = _.find(droptest, ['cname', _.capitalize(this.editedItem.cname)])
        if(findThis && _.capitalize(this.editedItem.cname) !=  _.capitalize(this.editedItem.dummycname) ) {
          return true
        } else {
          return false
        }
      },
      saveValid() {
        if( this.cateValid || this.editedItem.cname ==  '' || this.editedItem.description == '') {
          return true
        }
          return false
      },
      categoryData() {
        if(this.categorys.length > 1) {
          return _.orderBy(this.categorys, 'order');
        } else {
          return []
        }
      },
      userFData() {
         return this.$store.getters.userIndex
      },
    },
    methods: {
      isCashier(data) {
         var dataname = ''

         var getdata = firebase.database().ref('AccountUser/'+data)
         getdata.on('value', function(data) {
            dataname = data.val().fn + ' '
         })
         return dataname
      },
      isToday(date) {
         return moment().format('MMMM Do YYYY, h:mm:ss a')
      },
      getCategoryData() {
        let vm = this
        vm.categoryLoadingdata = true
        var categoryRef = firebase.database()
        var getCategory = categoryRef.ref('Category');
        getCategory.on('value', function(getData) {
          let keys = Object.keys(getData.val())
          vm.categorys = getData.val();
          keys.forEach( function (key) {
            vm.categorys[key].keyIndex = key
          })
          vm.categorys = _.orderBy(vm.categorys, 'order');
          vm.categoryLoadingdata = false
        })
      },
      addCategoryData() {
        let vm = this
        var categoryRef = firebase.database()
        var pushCategory = categoryRef.ref('Category').push()
        pushCategory.set({
          cname: _.capitalize(vm.editedItem.cname),
          description:  _.capitalize(vm.editedItem.description),
          total:  0
        }, function (error) {
          if(error) {
            alert('something is wrong')
          } else {
            vm.$refs.cnameRefs.focus()
            vm.snackbar = true
            vm.snackbarMessage = `Successfully added  ${_.capitalize(vm.editedItem.cname)}`
            vm.editedItem = { cname: '', description: '', keyIndex: ''}
          }
        })
      },
      newItem() {
        this.headlineTitle = 'Add category'
        this.dialogEdit = true
        this.$refs.cnameRefs.focus()
      },
      deleteDialog(item) {
        this.deletedialog = true
        this.deleteItem = {
          key : item.keyIndex,
          category: item.cname
          }

      },
      deleteItembtn() {
        let vm = this
        var categoryRef = firebase.database()
        categoryRef.ref(`Category/${vm.deleteItem.key}`).remove()
        
        var getProduct = firebase.database().ref(`Product`)
        getProduct.on('value', function(getData) {
          let keys = Object.keys(getData.val())
          var hold = _.mapValues(getData.val(), ['category', vm.deleteItem.category]);
          // console.log(getData.val())
          // var drop = _.takeWhile(hold, true)
          keys.forEach( function (key) {
            if(hold[key]) {
              firebase.database().ref(`Product/${key}`).remove()
            }
          })
        })

        vm.snackbar = true
        vm.snackbarMessage = `Successfully deleted`
        this.deletedialog = false
      },
      editItemSave(item) {
        let vm = this 
        var categoryRef = firebase.database()
        var updateCategory = categoryRef.ref(`Category/${item.keyIndex}`)
        updateCategory.update({
          cname: _.capitalize(vm.editedItem.cname),
          description: _.capitalize(vm.editedItem.description)
        }, function (error) {
          if(error) {
            alert('something is wrong')
          } else {
            vm.dialogEdit = false
            vm.snackbar = true
            vm.snackbarMessage = `Successfully Updated ${_.capitalize(vm.editedItem.cname)}`
            vm.editedItem = { cname: '', description: '', keyIndex: ''}
          }
        })
        if(vm.editedItem.cname != vm.editedItem.dummycname) {
          var getProduct = firebase.database().ref(`Product`)
          getProduct.on('value', function(getData) {
            let keys = Object.keys(getData.val())
            var hold = _.mapValues(getData.val(), ['category', vm.editedItem.dummycname]);
            // console.log(getData.val())
            // var drop = _.takeWhile(hold, true)
            keys.forEach( function (key) {
              if(hold[key]) {
                firebase.database().ref(`Product/${key}`).update({
                  category: vm.editedItem.cname
                })
              }
            })
          })
        } else {
          // do nothing jie 
        }

       
      },
      editItem(item) {
        let vm = this
        vm.$refs.cnameRefs.focus()
        this.headlineTitle = 'Edit category'
        this.dialogEdit = true
        this.editedItem = { cname: item.cname, dummycname: item.cname, description: item.description, keyIndex: item.keyIndex}
      },
      cancel() {
        this.dialogEdit = false
        this.editedItem = { cname: '', description:'', keyIndex: ''}
      }
    }
  }
</script>

<style lang="stylus" scoped>
  .my-event {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    border-radius: 2px;
    background-color: #009688;
    color: #ffffff;
    border: 1px solid #009688;
    width: 100%;
    font-weight: bold;
    font-size: 15px;
    padding: 10px;
    cursor: pointer;
    margin-bottom: 1px;
  }
</style>
        

