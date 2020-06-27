<template>
  <div class="centered">
    <div class="filter-select">
      Show
      <select v-model="show">
        <option v-for="(item, ind) in showOptions" :key="ind" :value="item">{{item}}</option>
      </select>
    </div>

    <div v-show="error" class="centered error-message">
      ERROR: {{error}}
    </div>

    <table class="centered display-table">
      <tr>
        <th>Name:</th>
        <th>checked In: </th>
        <th>Checked In At:</th>
        <th>Checked In By:</th>
        <th>Known Contacts</th>
        <th>Checked Out At:</th>
        <th>Checked Out By:</th>
        <th>Actions:</th>
      </tr>

      <tr v-for="(student, ind) in filteredList" :key="ind">
        <td>{{student.name}}</td>
        <td>{{isCheckedIn(student) ? 'Yes' : 'No'}}</td>
        <template v-if="student.checkedInAt">
          <td>{{student.checkedInAt | dateFormat}}</td>
          <td>{{student.checkedInBy}}</td>
        </template>
        <template v-else>
          <td>Not Checked in</td>
          <td>Not Checked in</td>          
        </template>
        <td>[{{student.knownContacts ? student.knownContacts.join(', ') : '' }}]</td>
        <template v-if="student.checkedOutAt">
          <td>{{student.checkedOutAt | dateFormat}}</td>
          <td>{{student.checkedOutBy}}</td>
        </template>
        <template v-else>
          <td>Not Checked Out</td>
          <td>Not Checked Out</td>          
        </template>

        <td>
          <input v-show="( !isCheckedIn(student) || (isCheckedIn(student) && !isCheckedOut(student)) )" type="text" v-model="contact[student.id]" placeholder="Contact" />
          <button v-if="!isCheckedIn(student)" @click.prevent="checkIn(student)">Check In</button>
          <button v-if="isCheckedIn(student) && !isCheckedOut(student)" @click.prevent="checkOut(student)">Check Out</button>
        </td>
      </tr>
    </table>

  </div>
</template>
<script>

import moment from 'moment';

//using constonts decreases the chance of mistakes due to typoes
const SHOW_ALL = "all";
const SHOW_CHECKED_IN = "Checked In";
const SHOW_NOT_CHECKED_IN = "Not Checked In";
const SHOW_CHECKED_OUT = "Checked Out";
const SHOW_NOT_CHECKED_OUT = "Not Checked Out";

const SHOW_OPTIONS = [SHOW_ALL, SHOW_CHECKED_IN, SHOW_NOT_CHECKED_IN, SHOW_CHECKED_OUT, SHOW_NOT_CHECKED_OUT];

export default {
  name: "StudentCheckIn",
  data() {
    return {
      students: [
        {  id: 1, name: "Mike", knownContacts: ['Bill', 'Jane'], checkedIn: false, checkkedInAt: null, checkedOutAt: null, checkedInBy: null, checkedOutBy: null },
        {  id: 2, name: "Joe",  knownContacts: ['Bob', 'Kelly', 'Todd'], checkedIn: false, checkedInAt: null, checkedOutAt: null, checkedInBy: null, checkedOutBy: null },
        {  id: 3, name: "Mary",  knownContacts: ['Heather', 'Jane'], checkedIn: false, checkkedInAt: null, checkedOutAt: null, checkedInBy: null, checkedOutBy: null },
        {  id: 4, name: "Jill",  knownContacts: ['Mary', 'Larry'], checkedIn: true, checkedInAt: null, checkedOutAt: null, checkedInBy: null, checkedOutBy: null }
      ],
      contact: {},
      showContact: false,
      show: SHOW_ALL,
      showOptions: SHOW_OPTIONS,
      error: ''
    };
  },
  computed: {
    filteredList() {
      let list = null;
      this.clearError();

      switch(this.show) {
        case SHOW_CHECKED_IN:
          //console.log("SHOW CHECKED IN");
          list = this.students.filter(item => this.isCheckedIn(item));
          break;
        case SHOW_NOT_CHECKED_IN:
          //console.log("SHOW NOT CHECKED IN");
          list = this.students.filter( (item) =>  !this.isCheckedIn(item)  );
          break;
        case SHOW_CHECKED_OUT:
          //console.log("SHOW CHECKED OUT");
          list = this.students.filter(item => this.isCheckedOut(item));
          break;
        case SHOW_NOT_CHECKED_OUT:
          //console.log("SHOW NOT CHECKED OUT");
          list = this.students.filter(item => !this.isCheckedOut(item));
          break;
        default:
          list = this.students;
      }

      return list;
    },
     contactEntered() {
      return (this.contact && this.contact.length > 1);
    },

  },
  methods: {
    setError(message) {
      this.error = message;
    },
    clearError() {
      this.error = '';
    },
    clearContact() {
      this.contact = {};
    },
    findIndexOf(id) {
      return this.students.findIndex( (student) => student.id === id );
    },
    hasContact(id) {
      if (!this.contact[id]) {
        this.setError("Please fill in the contact!");
        return false;
      } else {
        this.clearError();
        return true;
      }
    },
    updateStudent(student) {
      //The splice method is observed by Vue's reactivity and ensures the page will be updated when 
      //a student is updated and the filtered list will be updated.
      let index = this.findIndexOf(student.id);
      this.students.splice(index, 1, student);
    },
    isCheckedIn(student) {
      return !!student.checkedInAt;
    },
    isCheckedOut(student) {
      return !!student.checkedOutAt;
    },
    dateTime() {
      return new Date();
    },
    checkIn(student) {
      if (this.hasContact(student.id)) {
        student.checkedInBy = this.contact[student.id];
        student.checkedInAt = this.dateTime();
        this.updateStudent(student);
        this.clearContact();
      }
      
    },
    checkOut(student) {
      if (this.hasContact(student.id)) {
        //Check to see that the contact is in this students list of known contacts
        let contact = this.contact[student.id];
        if (student.knownContacts.find( (item) => item.toLowerCase() === contact.toLowerCase()) === undefined) {
          this.setError(`${this.contact} is not one of this student's known contacts`);
        } else {
          //Contact is a known contact so proceed with check out.
          student.checkedOutAt = this.dateTime();
          student.checkedOutBy = this.contact[student.id];
           this.updateStudent(student);
           this.clearContact();
           this.clearError();
        }
      }
    },
    toggle(student) {
      student.checkedIn = !student.checkedIn;
    }
  },
  filters: {
    dateFormat: function(value) {
      return moment(value).format('DD MM YYYY hh:mm A');
    }
  }
};
</script>
<style>
  table, th, td {
    border: 1px solid black;
  }

  td {
    padding: 1rem;
  }

  .centered {
    margin: auto;
  }

  .filter-select {
    margin-bottom: 1rem;
  }

  .error-message {
    margin-bottom: 1rem;
    color: red;
  }
</style>
