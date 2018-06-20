<!--

This component is used app-wide to set up the properties when making a new column.

Its usage might look something like this:

  <discoDefineNewColumn
    v-show="addNewColumn"
    :columnName="generatedColumnName"
    :columnLabel="generatedColumnLabel"
    @newColumnName="opData.newColumnName = $event"
    @newColumnLabel="opData.newColumnLabel = $event"
    @isValid="opData.isValid = $event"
  />

v-show: if the column name and label need to be persisted across visibility of this element and this state is not handled in the parent component, prefer v-show to v-if
columnName, columnLabel: these props are for the initial column name and label. These are different for each op so they should be generated parent-side as computed properties. See opOutlierDetection for a working example.
@newColumnName, @newColumnLabel: Emit events with the final column name and label that should be used in the op's data
@isValid: Emits boolean of current properties' collective validity

-->

<template>
<div>
  <label for="newColumnName" class="inputLabel caption">NEW COLUMN NAME</label>
  <el-input
    id="newColumnName"
    placeholder="Name of column"
    :value="newColumnName"
    clearable
    required
    :class="{ invalid: $v.newColumnName.$error, pulse: shouldPulse }"
    @blur="handleNewColumnInput($event, 'newColumnName')"
  />
  <p v-if="!$v.newColumnName.required" class="errorMessage caption">required field</p>
  <p v-else-if="!$v.newColumnName.patternMatch" class="errorMessage caption">
    Column names must start with a letter and cannot end with an underscore.
    <br />
    They may contain only lowercase alphanumeric characters and underscores.
  </p>

  <label for="newColumnLabel" class="inputLabel caption">NEW COLUMN LABEL</label>
  <el-input
    id="newColumnLabel"
    placeholder="Label of column"
    :value="newColumnLabel"
    clearable
    required
    :class="{ invalid: $v.newColumnLabel.$error, pulse: shouldPulse }"
    @blur="handleNewColumnInput($event, 'newColumnLabel')"
  />
  <p v-if="!$v.newColumnLabel.required" class="errorMessage caption">
    required field
  </p>
</div>
</template>

<script>
import { required } from 'vuelidate/lib/validators'

export default {
  validations() {
    return {
      newColumnName: {
        required,
        patternMatch: input => /^[a-z]([a-z0-9_]*[a-z0-9])?$/.test(input)  // all characters must be lowercase alphanumeric/underscore, first char must be letter, last cannot be underscore
      },
      newColumnLabel: { required }
    }
  },

  props: {
    columnName: {
      type: String,
      required: true
    },
    columnLabel: {
      type: String,
      required: true
    }
  },

  data() {
    return {
      newColumnName: this.columnName,
      newColumnLabel: this.columnLabel,

      unwatchValidation: '',
      shouldAutocorrect: true,
      shouldPulse: false
    }
  },

  created() {
    // Emit boolean reflecting current validation state:
    this.unwatchValidation = this.$watch('$v.$invalid', e => {
      this.$emit('isValid', !e)
    })
  },

  beforeDestroy() {
    this.unwatchValidation()
  },

  watch: {
    columnName(newName) {
      if(!this.$v.newColumnName.$dirty) {
        this.newColumnName = newName
        this.emitNewName()
      }
    },
    columnLabel(newLabel) {
      if(!this.$v.newColumnLabel.$dirty) {
        this.newColumnLabel = newLabel
        this.emitNewLabel()
      }
    }
  },

  methods: {
    autoCorrectNewColumn() {
      if(this.shouldAutocorrect && this.$v.$anyDirty) {
        // Ensure this function runs only once when the user touched neither new column field
        this.shouldAutocorrect = false

        // If it's the name that's been changed,
        if(this.$v.newColumnName.$dirty && !this.$v.newColumnName.patternMatch) {
          // And the user hasn't changed it to an empty string
          if (this.$v.newColumnName.$model) {
            // Trigger pulse animations to signal to the user some change is happening:
            this.shouldPulse = true

            // Set the column LABEL to the column NAME's invalid input:
            this.$v.newColumnLabel.$model = this.$v.newColumnName.$model

            // Autocorrect newColumnName:
            this.$v.newColumnName.$model = this.$v.newColumnName.$model.toLowerCase()  // must be lowercase
                                                           .replace(/ +/g, '_')          // spaces become underscores
                                                           .replace(/[^a-z\d_]/g, '')    // letters, numbers, underscores
                                                           .replace(/^[^a-z]*/, '')      // must start with a letter
                                                           .replace(/_*$/, '')           // may not end with underscore

            this.emitAllInput()
          }
        }
      } else {
        // Remove pulse animations for all future checks:
        this.shouldPulse = false
      }
    },

    emitAllInput() {
      this.emitNewName()
      this.emitNewLabel()
    },

    emitNewLabel() {
      this.$emit('newColumnLabel', this.newColumnLabel)
    },

    emitNewName() {
      this.$emit('newColumnName', this.newColumnName)
    },

    handleNewColumnInput(e, inputField) {
      const newInput = e.target.value

      this[inputField] = newInput
      this.$v[inputField].$touch()

      this.autoCorrectNewColumn()

      // emit new name or label
      this.$emit(inputField, this[inputField])
    }
  }
}
</script>

<style lang="scss" scoped>
.maxWidth {
  width: 100%;
}

.inputLabel {
  display: block;
  padding-top: 36px;
  padding-bottom: 6px;
  text-transform: uppercase;
}

.invalid {
  /deep/ .el-input__inner {
    border: 1px solid pink;
  }
}

.errorMessage {
  color: pink;
  position: absolute;
  margin-top: 3px;
}

// Pulse animations:
.pulse {
  animation: pulse 1s 1;  // Run only one time
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(pink, 0.4)
  }
  70% {
      box-shadow: 0 0 0 10px rgba(pink, 0);
  }
  100% {
      box-shadow: 0 0 0 0 rgba(pink, 0);
  }
}
</style>