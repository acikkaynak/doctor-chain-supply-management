<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">Parola Sıfırla</div>

                    <div class="card-body">
                        <div class="row">
                            <div class="col-12">
                                <BackendAndFrontendCombined :errors="validation.errors" :message="validation.message" :show="backendAndFrontendCombinedErrorsStatus" :validation-attributes="validation.validationAttributes" :show-header-message="validation.showHeaderMessage" :vuelidate="vuelidate$" />
                            </div>

                        </div>

                        <div class="row mb-3">
                            <label for="email" class="col-md-4 col-form-label text-md-end">{{ $t('modules.auth.register.form.email.title') }}</label>

                            <div class="col-md-6">
                                <input id="email" v-model="email" type="email" class="form-control" name="email" required autocomplete="email" autofocus>
                            </div>
                        </div>

                        <div class="row mb-0">
                            <div class="col-md-8 offset-md-4">
                                <button type="button" @click.prevent="sendResetLink" class="btn btn-primary">Parola Yenileme Linki Gönder</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>


<script>
import BackendAndFrontendCombined from "@/src/components/ValidationMessages/BackendAndFrontendCombined.vue";
import useVuelidate from '@vuelidate/core'
import { required, minLength, maxLength, email } from '@/src/helpers/TranslatedVuelidateValidators.js'
import emitter from '@/EventBus.js'

export default {
    name: "Auth.Password.Email",
    data(){
        return {
            form_is_posted: false,

            email: null,

            validation: {
                errors: [],
                message: '',
                validationAttributes: {
                    email: this.$t('modules.auth.login.form.email.title'),
                },
                show_backend_and_frontend_combined_error_messages: true,
                showHeaderMessage: false,
            },
        }
    },
    components: {
        BackendAndFrontendCombined,
    },
    computed: {
        backendAndFrontendCombinedErrorsStatus() {
            return (
                this.form_is_posted &&
                this.validation
                    .show_backend_and_frontend_combined_error_messages
            )
        },
    },
    created() {
        emitter.emit('set-title', 'Parola Yenileme İsteğinde Bulun')
    },
    setup() {
        return { vuelidate$: useVuelidate() }
    },
    validations() {
        return {
            email: {
                email,
                maxLength: maxLength(60),
                required,
                $autoDirty: true,
                $lazy: true,
            },
        }
    },
    methods: {
        async sendResetLink(e){
            const $this = this
            e.preventDefault()

            this.vuelidate$.$reset()
            this.validation.errors = []
            this.validation.message = ''

            this.form_is_posted = true
            $this.vuelidate$.$touch()
            if ($this.vuelidate$.$pending || $this.vuelidate$.$error) return

            const result = await $this.vuelidate$.$validate()
            if (!result) {
                return
            }

            const data = {
                email: this.email,
            }
            await axios.get('/sanctum/csrf-cookie').then(async () => {
                await axios.post('/api/auth/password/email', data)
                    .then((response)=>{
                        this.$swal
                            .fire(response.data.message.title, response.data.message.body, response.data.message.type)
                            .then(() => {
                                if (response.data.status){
                                    this.$router.push({name:"Auth.Login"})
                                }
                            })
                    })
                    .catch((e) => {
                        if (e.response.status === 422) {
                            this.validation.errors = e.response.data.errors
                            this.validation.message = e.response.data.message
                        } else {
                            this.$swal.fire(e.response.data.message.title, e.response.data.message.body, e.response.data.message.type)
                        }
                    })
            })
        },
    }
}
</script>

<style scoped>

</style>
