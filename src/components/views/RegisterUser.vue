<template>
  <div>
    <h1>{{ $t('registerUser.title') }}</h1>

    <v-form
      v-if="newUser==null"
      id="registerForm"
      ref="form"
      v-model="validForm"
      @submit.prevent>

      <v-text-field
        id="email"
        v-model="email"
        :label="$t('registerUser.email')"
        :placeholder="$t('registerUser.emailPlaceholder')"
      />

      <v-text-field
        id="username"
        v-model="ctx.user.username"
        :rules="[rules.required]"
        :label="$t('registerUser.username')"
      />

      <Password
        v-model="password"
        :confirmation="true"/>

      <v-autocomplete
        id="hosting"
        v-model="hosting"
        :items="hostingsSelection"
        :rules="[rules.required]"
        :label="$t('registerUser.hosting')"
        :placeholder="$t('registerUser.hostingPlaceholder')"
      />

      <v-btn
        id="submitButton"
        :disabled="!validForm||submitting"
        @click="submit"
      >{{ $t('registerUser.create') }}</v-btn>

      <v-btn
        id="clearButton"
        @click="clear"
      >{{ $t('registerUser.clear') }}</v-btn>

      <div>
        By registering you agree with our
        <a
          target="_blank"
          href="https://pryv.com/terms-of-use/">
        terms of use</a>.
      </div>
    </v-form>
    <div v-if="ctx.isAccessRequest()">
      <v-divider class="mt-3 mb-2"/>
      <router-link :to="{ name: 'Authorization' }"><h3>{{ $t('registerUser.goToSignIn') }}</h3></router-link>
    </div>

    <Alerts
      :successMsg="success"
      :errorMsg="error"/>
  </div>
</template>

<script>
import i18n from '../../i18n';
import Password from './bits/Password.vue';
import Alerts from './bits/Alerts.vue';
import Context from '../../context.js';
import controllerFactory from '../controller/controller.js';

export default {
  components: {
    Password,
    Alerts,
  },
  data: () => ({
    password: '',
    email: '',
    hosting: '',
    hostingsSelection: [],
    newUser: null,
    submitting: false,
    ctx: {},
    c: null,
    error: '',
    success: '',
    rules: {
      required: value => !!value || i18n.t('global.requiredField'),
    },
    validForm: false,
  }),
  async created () {
    this.ctx = new Context(this.$route.query);
    await this.ctx.init();
    this.c = controllerFactory(this.ctx);
    // Fill selector with available hostings, preselect first one
    this.c.loadHostings()
      .then(this.initHostings)
      .catch(this.showError);
  },
  methods: {
    submit () {
      if (this.$refs.form.validate()) {
        this.submitting = true;

        this.generateRandomEmailIfNeeded();
        // get least occupied core in the hosting
        let availableCore = '';
        this.hostingsSelection.forEach(selection => {
          if (selection.value === this.hosting) {
            availableCore = selection.availableCore;
          }
        });
        this.c.createUser(availableCore, this.password, this.email, this.hosting)
          .then((newUser) => {
            this.newUser = newUser;
            this.success = i18n.t('registerUser.registerSuccess', {username: newUser.username});
            if (!this.ctx.isAccessRequest()) {
              location.href = this.ctx.pryvService.apiEndpointForSync(newUser.username);
            }
          })
          .catch(this.showError)
          .finally(() => { this.submitting = false; });
      }
    },
    clear () {
      this.$refs.form.reset();
    },
    initHostings (hostings) {
      this.hostingsSelection = hostings;
      if (this.hostingsSelection.length > 0) {
        this.hosting = this.hostingsSelection[0].value;
      }
    },
    showError (error) {
      this.error = error.msg;
    },
    generateRandomEmailIfNeeded () {
      if (this.email == null || (typeof this.email === 'string' && this.email.length === 0)) {
        this.email = randomUsername(20) + '@pryv.io';
      }
    },
  },
};

function randomUsername (length) {
  let result = '';
  let characters = 'abcdefghijklmnopqrstuvwxyz0123456789';
  let charactersLength = characters.length;
  for (let i = 0; i < length; i++) {
    result += characters.charAt(Math.floor(Math.random() * charactersLength));
  }
  return result;
}
</script>
