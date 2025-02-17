<template>
  <v-card>
    <v-card-title class="green darken-3">
      <div class="headline">
        <span class="white--text">
          Subject Information
        </span>
        <span v-if="existingSubject" class="grey--text text--lighten-1">
          {{ `#${subject.id}` }}
        </span>
      </div>
    </v-card-title>
    <v-card-text>
      <v-layout wrap column>
        <v-radio-group
          row
          v-model="radioGroup"
          v-if="!(existingSubject || createMode)"
        >
          <v-radio label="New Subject" value="new" />
          <v-radio label="Existing subject" value="existing" />
        </v-radio-group>
        <v-layout row pl-3 v-if="radioGroup == 'existing'">
          <v-flex>
            <v-select
              v-model="selectedSubjectRepresentation"
              :items="
                subjects.map(subject => getSubjectRepresentation(subject))
              "
            />
          </v-flex>
        </v-layout>
        <v-form v-if="radioGroup == 'new'" @submit.prevent="submit">
          <v-text-field
            v-model="subject.firstName"
            label="First Name"
            :counter="64"
            :disabled="!(editable || subject.firstName)"
            :readonly="!editable"
          />
          <v-text-field
            v-model="subject.lastName"
            label="Last Name"
            :counter="64"
            :disabled="!(editable || subject.lastName)"
            :readonly="!editable"
          />
          <v-select
            v-model="subjectDominantHand"
            label="Dominant Hand"
            :items="Object.keys(dominantHandOptions)"
            :readonly="!editable"
            :disabled="!(editable || subjectDominantHand)"
          />
          <v-select
            v-model="subjectSex"
            label="Sex"
            :items="Object.keys(sexOptions)"
            :readonly="!editable"
            :disabled="!(editable || subjectSex)"
          />
          <v-spacer />
          <v-select
            v-model="subjectGender"
            label="Gender"
            :items="Object.keys(genderOptions)"
            :readonly="!editable"
            :disabled="!(editable || subjectGender)"
          />
          <v-menu
            v-model="dob_menu"
            :close-on-content-click="false"
            :nudge-right="40"
            lazy
            transition="scale-transition"
            offset-y
            full-width
            min-width="290px"
          >
            <template v-slot:activator="{ on }">
              <v-text-field
                v-model="formattedDate"
                label="Date of Birth"
                prepend-icon="cake"
                :disabled="!(editable || formattedDate)"
                :readonly="!editable"
                v-on="on"
              ></v-text-field>
            </template>
            <v-date-picker
              v-model="subject.dateOfBirth"
              @input="dob_menu = false"
            ></v-date-picker>
          </v-menu>
        </v-form>
      </v-layout>
    </v-card-text>
    <v-card-actions>
      <v-flex shrink px-3>
        <v-switch
          v-if="existingSubject"
          v-model="editable"
          :label="editable ? 'Edit Mode' : 'View Mode'"
        />
      </v-flex>
      <v-spacer />
      <v-btn
        flat
        v-if="editable && existingSubject"
        color="warning"
        @click="updateExistingSubject()"
      >
        Update
      </v-btn>
      <v-btn
        flat
        v-if="!existingSubject && radioGroup == 'new'"
        color="success"
        @click="createNewSubject"
      >
        Create
      </v-btn>
      <v-btn color="green darken-1" flat @click="closeDialog">
        Cancel
      </v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
import { mapActions, mapState } from 'vuex'
import { sexOptions, genderOptions, dominantHandOptions } from './choices.js'
import { getKeyByValue } from './utils.js'

const cleanSubject = {
  idNumber: '',
  firstName: '',
  lastName: '',
  dateOfBirth: null,
  dominantHand: null,
  sex: null,
  gender: null
}

export default {
  name: 'SubjectInfoCard',
  props: {
    existingSubject: Object,
    createMode: { type: Boolean, default: false }
  },
  created() {
    if (this.existingSubject) {
      this.subject = Object.assign({}, this.existingSubject)
      this.editable = false
    }
  },
  data: () => ({
    selectedSubjectRepresentation: '',
    subject: Object.assign({}, cleanSubject),
    editable: true,
    dob_menu: false,
    radioGroup: 'new',
    sexOptions,
    genderOptions,
    dominantHandOptions
  }),
  computed: {
    subjectSex: {
      get: function() {
        return getKeyByValue(this.sexOptions, this.subject.sex)
      },
      set: function(newValue) {
        this.subject.sex = this.sexOptions[newValue]
      }
    },
    subjectGender: {
      get: function() {
        return getKeyByValue(this.genderOptions, this.subject.gender)
      },
      set: function(newValue) {
        this.subject.gender = this.genderOptions[newValue]
      }
    },
    subjectDominantHand: {
      get: function() {
        return getKeyByValue(
          this.dominantHandOptions,
          this.subject.dominantHand
        )
      },
      set: function(newValue) {
        this.subject.dominantHand = this.dominantHandOptions[newValue]
      }
    },
    formattedDate: function() {
      return formatDate(this.subject.dateOfBirth)
    },
    ...mapState('research', ['subjects'])
  },
  methods: {
    closeDialog() {
      if (this.existingSubject && this.editable) this.editable = false
      this.$emit('close-subject-dialog')
    },
    updateExistingSubject() {
      this.updateSubject(this.subject)
        .then(data => (this.subject = Object.assign({}, data)))
        .then(this.closeDialog())
    },
    createNewSubject() {
      this.createSubject(this.subject)
        .then((this.editable = false))
        .then(this.closeDialog())
    },
    getSubjectRepresentation(subject) {
      let rep = `Subject #${subject.id}`
      if (subject.firstName && subject.lastName) {
        rep = rep.concat(` [${subject.firstName} ${subject.lastName}]`)
      }
      return rep
    },
    ...mapActions('research', ['createSubject', 'updateSubject'])
  },
  watch: {
    editable: function(isEditable) {
      if (!isEditable) {
        this.subject = Object.assign({}, this.existingSubject)
      }
    }
  }
}

function formatDate(date) {
  if (!date) return null
  const [year, month, day] = date.split('-')
  return `${day}/${month}/${year}`
}
</script>

<style lang="scss" scoped>
>>> label {
  margin-bottom: 0rem;
}
</style>
