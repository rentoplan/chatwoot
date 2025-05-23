<script>
import { required, minLength } from '@vuelidate/validators';
import { mapGetters } from 'vuex';
import { useVuelidate } from '@vuelidate/core';
import { useAlert } from 'dashboard/composables';
import NextButton from 'dashboard/components-next/button/Button.vue';

export default {
  components: {
    NextButton,
  },
  props: {
    show: {
      type: Boolean,
      default: false,
    },
    hasAccounts: {
      type: Boolean,
      default: true,
    },
  },
  emits: ['closeAccountCreateModal'],
  setup() {
    return { v$: useVuelidate() };
  },
  data() {
    return {
      accountName: '',
    };
  },
  validations() {
    return {
      accountName: {
        required,
        minLength: minLength(1),
      },
    };
  },
  computed: {
    ...mapGetters({
      uiFlags: 'agents/getUIFlags',
    }),
  },
  methods: {
    async addAccount() {
      try {
        const account_id = await this.$store.dispatch('accounts/create', {
          account_name: this.accountName,
        });
        this.$emit('closeAccountCreateModal');
        useAlert(this.$t('CREATE_ACCOUNT.API.SUCCESS_MESSAGE'));
        window.location = `/app/accounts/${account_id}/dashboard`;
      } catch (error) {
        if (error.response.status === 422) {
          useAlert(this.$t('CREATE_ACCOUNT.API.EXIST_MESSAGE'));
        } else {
          useAlert(this.$t('CREATE_ACCOUNT.API.ERROR_MESSAGE'));
        }
      }
    },
  },
};
</script>

<template>
  <woot-modal :show="show" :on-close="() => $emit('closeAccountCreateModal')">
    <div class="flex flex-col h-auto overflow-auto">
      <woot-modal-header
        :header-title="$t('CREATE_ACCOUNT.NEW_ACCOUNT')"
        :header-content="$t('CREATE_ACCOUNT.SELECTOR_SUBTITLE')"
      />
      <div v-if="!hasAccounts" class="mx-8 mt-6 mb-0 text-sm">
        <div class="flex items-center rounded-md alert">
          <div class="ml-1 mr-3">
            <fluent-icon icon="warning" />
          </div>
          {{ $t('CREATE_ACCOUNT.NO_ACCOUNT_WARNING') }}
        </div>
      </div>

      <form class="flex flex-col w-full" @submit.prevent="addAccount">
        <div class="w-full">
          <label :class="{ error: v$.accountName.$error }">
            {{ $t('CREATE_ACCOUNT.FORM.NAME.LABEL') }}
            <input
              v-model="accountName"
              type="text"
              :placeholder="$t('CREATE_ACCOUNT.FORM.NAME.PLACEHOLDER')"
              @input="v$.accountName.$touch"
            />
          </label>
        </div>
        <div class="w-full flex justify-end gap-2 items-center">
          <NextButton
            faded
            slate
            type="reset"
            :label="$t('CREATE_ACCOUNT.FORM.CANCEL')"
            @click.prevent="() => $emit('closeAccountCreateModal')"
          />
          <NextButton
            type="submit"
            :label="$t('CREATE_ACCOUNT.FORM.SUBMIT')"
            :is-loading="uiFlags.isCreating"
            :disabled="
              v$.accountName.$invalid ||
              v$.accountName.$invalid ||
              uiFlags.isCreating
            "
          />
        </div>
      </form>
    </div>
  </woot-modal>
</template>
