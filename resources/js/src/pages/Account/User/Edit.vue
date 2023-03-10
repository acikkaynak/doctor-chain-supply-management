<template>
    <div class="row justify-content-center">
        <div class="col-12">
            <template v-if="user.isLoading">Kullanıcı verisi alınıyor..</template>
            <template v-else>
                <template v-if="user.data.status === false">
                    <div class="alert alert-danger">{{ user.data.message.body }}</div>
                </template>
                <template v-else>
                    <div class="card">
                        <div class="card-header">{{ $t('modules.account.edit.title') }}</div>

                        <div class="card-body">
                            <div class="row">
                                <div class="col-12">
                                    <BackendAndFrontendCombined :errors="validation.errors" :message="validation.message" :show="backendAndFrontendCombinedErrorsStatus" :validation-attributes="validation.validationAttributes" :show-header-message="validation.showHeaderMessage" :vuelidate="vuelidate$" />
                                </div>

                            </div>

                            <div class="row mb-3">
                                <label for="first_name" class="col-md-4 col-form-label text-md-end">{{ $t('modules.auth.register.form.first_name.title') }}</label>

                                <div class="col-md-6">
                                    <input
                                        id="first_name"
                                        type="text"
                                        v-model="first_name"
                                        class="form-control"
                                        name="first_name"
                                        :placeholder="$t('modules.auth.register.form.first_name.placeholder')"
                                        :class="{
                                    'is-invalid': vuelidate$.first_name.$error,
                                    'is-valid': vuelidate$.first_name.$dirty && !vuelidate$.first_name.$invalid,
                                }"
                                        @input="vuelidate$.first_name.$touch()"
                                        @focus="vuelidate$.first_name.$touch()"
                                    >
                                    <SingleInputError :vuelidate-object="vuelidate$.first_name" />
                                </div>
                            </div>

                            <div class="row mb-3">
                                <label for="last_name" class="col-md-4 col-form-label text-md-end">{{ $t('modules.auth.register.form.last_name.title') }}</label>

                                <div class="col-md-6">
                                    <input
                                        id="last_name"
                                        type="text"
                                        v-model="last_name"
                                        class="form-control"
                                        name="last_name"
                                        :placeholder="$t('modules.auth.register.form.last_name.placeholder')"
                                        :class="{
                                    'is-invalid': vuelidate$.last_name.$error,
                                    'is-valid': vuelidate$.last_name.$dirty && !vuelidate$.last_name.$invalid,
                                }"
                                        @input="vuelidate$.last_name.$touch()"
                                        @focus="vuelidate$.last_name.$touch()"
                                    >
                                    <SingleInputError :vuelidate-object="vuelidate$.last_name" />
                                </div>
                            </div>

                            <div class="row mb-3">
                                <label for="specialty" class="col-md-4 col-form-label text-md-end">{{ $t('modules.recipe.form.specialty.title') }}</label>

                                <div class="col-md-6">
                                    <select
                                        id="specialty"
                                        v-model="specialty"
                                        class="form-control"
                                        name="specialty"
                                        :class="{
                                    'is-invalid': vuelidate$.specialty.$error,
                                    'is-valid': vuelidate$.specialty.$dirty && !vuelidate$.specialty.$invalid,
                                }"
                                        @input="vuelidate$.specialty.$touch()"
                                        @focus="vuelidate$.specialty.$touch()"
                                    >
                                        <option :value="null">{{ $t('general.select') }}</option>
                                        <option v-for="specialtiesItem in getSpecialtiesCategories" :value="specialtiesItem.id" :key="'specialty_' + specialtiesItem.id">{{ specialtiesItem.name }}</option>
                                    </select>
                                    <SingleInputError :vuelidate-object="vuelidate$.specialty" />
                                </div>
                            </div>

                            <div class="row mb-0">
                                <div class="col-md-6 offset-md-4">
                                    <button type="button" class="btn btn-danger me-2" @click.prevent="reset">{{ $t('general.reset') }}</button>
                                    <button type="button" class="btn btn-primary" @click.prevent="update">{{ $t('general.update') }}</button>
                                </div>
                            </div>
                        </div>
                    </div>
                </template>
            </template>
        </div>
    </div>
</template>

<script>
import AddressForm from "@/src/components/Address/Form.vue";
import BackendAndFrontendCombined from "@/src/components/ValidationMessages/BackendAndFrontendCombined.vue";
import SingleInputError from "@/src/components/ValidationMessages/SingleInputError.vue";
import useVuelidate from '@vuelidate/core'
import emitter from '@/EventBus.js'
import { required, minLength, maxLength } from '@/src/helpers/TranslatedVuelidateValidators.js'
import {mapActions,mapGetters} from "vuex";

export default {
    name: "Account.Edit",
    components: {
        BackendAndFrontendCombined,
        SingleInputError,
        AddressForm,
    },
    data(){
        return {
            form_is_posted: false,

            user: {
                isLoading: true,
                data: null,
            },

            first_name: null,
            last_name: null,
            specialty: null,

            specialties: [],

            validation: {
                errors: [],
                message: '',
                validationAttributes: {
                    first_name: this.$t('modules.auth.register.form.first_name.title'),
                    last_name: this.$t('modules.auth.register.form.last_name.title'),
                    specialty: this.$t('modules.auth.register.form.specialty.title'),
                },
                show_backend_and_frontend_combined_error_messages: true,
                showHeaderMessage: false,
            },
        }
    },
    computed: {
        backendAndFrontendCombinedErrorsStatus() {
            return (
                this.form_is_posted &&
                this.validation
                    .show_backend_and_frontend_combined_error_messages
            )
        },
        ...mapGetters('global',[
            'getSpecialtiesCategories'
        ]),
    },
    created() {
        this.prepareUserData()
        emitter.emit('set-title', 'Hesabımı Düzenle')
    },
    setup() {
        return { vuelidate$: useVuelidate() }
    },
    validations() {
        return {
            first_name: {
                minLength: minLength(2),
                maxLength: maxLength(30),
                required,
                $autoDirty: true,
                $lazy: true,
            },
            last_name: {
                minLength: minLength(2),
                maxLength: maxLength(30),
                required,
                $autoDirty: true,
                $lazy: true,
            },
            specialty: {
                required,
                $autoDirty: true,
                $lazy: true,
            },
        }
    },
    methods: {
        ...mapActions('auth', ['setUser']),
        reset() {
            this.first_name = this.user.data.data.first_name
            this.last_name = this.user.data.data.last_name
            this.specialty = this.user.data.data.specialty ? this.user.data.data.specialty.id : null

            this.vuelidate$.$reset()
            this.form_is_posted = false
        },
        async prepareUserData(){
            const $this = this
            $this.form_is_loading = true
            await axios
                .get('/api/account/profile')
                .then((response) => {
                    $this.user.isLoading = false
                    $this.user.data = response.data
                    $this.reset()
                })
                .catch((e) => {
                    $this.form_is_loading = false
                    this.$swal.fire(e.response.data.message.title, e.response.data.message.body, e.response.data.message.type)
                })
        },
        async update(e) {
            const $this = this
            e.preventDefault()

            this.form_is_posted = true
            $this.vuelidate$.$touch()
            if ($this.vuelidate$.$pending || $this.vuelidate$.$error) return

            const result = await $this.vuelidate$.$validate()
            if (!result) {
                return
            }

            const data = {
                first_name: this.first_name,
                last_name: this.last_name,
                specialty: this.specialty,
            }

            $this.$swal.fire({
                title: this.$t('modules.account.edit.are_you_sure'),
                showCancelButton: true,
            }).then(async (result) => {
                if (result.isConfirmed) {
                    await $this.axios.put('/api/account/profile', data)
                        .then((response) => {
                            if (response.data.status) {
                                this.setUser()
                                $this.vuelidate$.$reset()
                            }
                            this.$swal.fire(response.data.message.title, response.data.message.body, response.data.message.type)
                        })
                        .catch((e) => {
                            if (e.response.status === 422) {
                                this.validation.errors = e.response.data.errors
                                this.validation.message = e.response.data.message
                            } else {
                                this.$swal.fire(e.response.data.message.title, e.response.data.message.body, e.response.data.message.type)
                            }
                        })
                }
            })
        },
    }
}
</script>

<style scoped>

</style>
