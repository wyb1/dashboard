<!--
Copyright 2018 by The Gardener Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
 -->

 <template>
  <secret-dialog
    :value=value
    :data="secretData"
    :dataValid="valid"
    :secret="secret"
    cloudProviderKind="gcp"
    color="green"
    infraIcon="mdi-google"
    backgroundSrc="/static/background_gcp.svg"
    createTitle="Add new Google Secret"
    replaceTitle="Replace Google Secret"
    @input="onInput">

    <template slot="data-slot">
      <v-layout row>
        <v-flex xs12>
          <v-text-field
            ref="serviceAccountKey"
            color="green"
            v-model="serviceAccountKey"
            :label="serviceAccountKeyLabel"
            :error-messages="getErrorMessages('serviceAccountKey')"
            @input="$v.serviceAccountKey.$touch()"
            @blur="$v.serviceAccountKey.$touch()"
            textarea
            multi-line
            hint="Enter or drop a service account key in JSON format"
            persistent-hint
          ></v-text-field>
        </v-flex>
      </v-layout>
    </template>

  </secret-dialog>

</template>


<script>
  import SecretDialog from '@/dialogs/SecretDialog'
  import { required } from 'vuelidate/lib/validators'
  import { serviceAccountKey } from '@/utils/validators'
  import { handleTextFieldDrop, getValidationErrors, setDelayedInputFocus } from '@/utils'

  const validationErrors = {
    serviceAccountKey: {
      required: 'You can\'t leave this empty.',
      serviceAccountKey: 'Not a valid Service Account Key'
    }
  }

  export default {
    components: {
      SecretDialog
    },
    props: {
      value: {
        type: Boolean,
        required: true
      },
      secret: {
        type: Object
      }
    },
    data () {
      return {
        serviceAccountKey: undefined,
        validationErrors
      }
    },
    validations () {
      // had to move the code to a computed property so that the getValidationErrors method can access it
      return this.validators
    },
    computed: {
      valid () {
        return !this.$v.$invalid
      },
      secretData () {
        return {
          project: this.projectId,
          'serviceaccount.json': this.serviceAccountKey
        }
      },
      validators () {
        const validators = {
          serviceAccountKey: {
            required,
            serviceAccountKey
          }
        }
        return validators
      },
      isCreateMode () {
        return !this.secret
      },
      projectId () {
        try {
          const key = JSON.parse(this.serviceAccountKey)
          const projectId = key.project_id ? key.project_id : ''
          return projectId
        } catch (err) {
          return ''
        }
      },
      serviceAccountKeyLabel () {
        return this.isCreateMode ? 'Service Account Key' : 'New Service Account Key'
      }
    },
    methods: {
      onInput (value) {
        this.$emit('input', value)
      },
      reset () {
        this.$v.$reset()

        this.serviceAccountKey = ''

        if (!this.isCreateMode) {
          setDelayedInputFocus(this, 'serviceAccountKey')
        }
      },
      getErrorMessages (field) {
        return getValidationErrors(this, field)
      }
    },
    watch: {
      value: function (value) {
        if (value) {
          this.reset()
        }
      }
    },
    mounted () {
      handleTextFieldDrop(this.$refs.serviceAccountKey, /json/)
    }
  }
</script>


<style lang="styl" scoped>
  .gce_credential {
    >>> .input-group--textarea textarea {
      font-family: monospace;
      font-size: 14px;
    }

    >>> .input-group--text-field.input-group--textarea:not(.input-group--full-width) .input-group__input {
       border: 1px solid rgba(0,0,0,0.3);
       background-color: rgba(0,20,0,0.02);
    }
  }
</style>
