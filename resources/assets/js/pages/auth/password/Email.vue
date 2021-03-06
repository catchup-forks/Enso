<template>

    <div class="column box login">
        <h3 class="title is-3 has-text-black has-text-centered has-margin-bottom-medium">
            <figure class="image is-24x24 logo">
                <img src="/images/logo.svg"/>
            </figure>
            {{ appName }}
        </h3>
        <form class="has-margin-bottom-medium"
            @submit.prevent="submit()">
            <div class="field">
                <div class="control has-icons-left has-icons-right">
                    <input class="input"
                        :class="{ 'is-danger': hasErrors, 'is-success': isSuccessful }"
                        type="email"
                        placeholder="Email"
                        v-model="email">
                    <span class="icon is-small is-left">
                        <fa icon="envelope"></fa>
                    </span>
                    <span class="icon is-small is-right has-text-success"
                        v-if="isSuccessful">
                        <fa icon="check"></fa>
                    </span>
                    <span class="icon is-small is-right has-text-danger"
                        v-if="hasErrors">
                        <fa icon="exclamation-triangle"></fa>
                    </span>
                </div>
            </div>
            <div class="field">
                <button class="button is-primary is-fullwidth"
                    :class="{ 'is-loading': loading }"
                    type="submit"
                    @click.prevent="submit()">
                    <span class="icon is-small">
                        <fa icon="user"></fa>
                    </span>
                    <span>Send a reset passworkd link</span>
                </button>
            </div>
        </form>
    </div>
</template>

<script>

import fontawesome from '@fortawesome/fontawesome';
import {
    faEnvelope, faCheck, faExclamationTriangle, faUser,
} from '@fortawesome/fontawesome-free-solid/shakable.es';

fontawesome.library.add(faEnvelope, faCheck, faExclamationTriangle, faUser);

export default {
    name: 'Email',

    props: {
        appName: {
            type: String,
            required: true,
        },
    },

    data() {
        return {
            loading: false,
            email: null,
            hasErrors: false,
            isSuccessful: false,
        };
    },

    watch: {
        email: {
            handler() {
                this.hasErrors = false;
            },
        },
    },

    methods: {
        submit() {
            this.loading = true;
            this.isSuccessful = false;
            this.hasErrors = false;

            axios.post('/api/password/email', { email: this.email }).then(({ data }) => {
                this.loading = false;
                this.isSuccessful = true;
                this.$toastr.success(data.status);
            }).catch((error) => {
                this.loading = false;
                this.hasErrors = true;

                const { status, data } = error.response;

                if (status === 422) {
                    this.reportLoginError(data);
                    return;
                }

                throw error;
            });
        },
        reportLoginError(data) {
            if (data.errors) {
                if (data.errors.email) {
                    this.$toastr.error(data.errors.email[0]);
                }

                return;
            }

            this.$toastr.error(data.message);
        },
    },
};

</script>
