<template>
  <div>
    <v-text-field
      :id="id"
      :value="value"
      :appendIcon="visiblePass ? 'lock_open' : 'lock'"
      :type="visiblePass ? 'text' : 'password'"
      :rules="[rules.required]"
      :label="labelPassword"
      @input="$emit('input', $event)"
      @click:append="() => (visiblePass = !visiblePass)"/>

    <v-text-field
      v-if="confirmation"
      :id="`${id}Confirmation`"
      v-model="repass"
      :type="visiblePass ? 'text' : 'password'"
      :rules="[matchPassword]"
      :label="labelPasswordConfirmation"/>
  </div>
</template>

<script>
import i18n from '../../../i18n';

export default {
  props: {
    confirmation: {type: Boolean, default: false},
    value: {type: String, default: ''},
    labelPassword: {type: String, default: i18n.t('password.password')},
    labelPasswordConfirmation: {type: String, default: i18n.t('password.passwordConfirmation')},
    id: {type: String, default: 'password'},
  },
  data: () => ({
    visiblePass: false,
    repass: '',
    rules: {
      required: value => !!value || i18n.t('password.passwordIsRequired'),
    },
  }),
  computed: {
    matchPassword: function () {
      return this.value === this.repass || i18n.t('password.passwordConfirmationNotMatch');
    },
  },
};
</script>
