<template>
    <f7-page class="login-page" :color="theme.name_color_theme" :class="theme.class_bg_body">
        <!-- Sección del Fondo -->
        <div class="login-background"></div>
        
        <f7-block class="no-margin-vertical padding-vertical" :class="theme.class_bg_header" :color="theme.name_color_theme">
            <!-- Logo -->
            <f7-row class="footer-logo">
                <f7-col width="1">
                    <logo heightLogo="40" alternativePercentage="25%" class="display-flex justify-content-center"></logo>
                </f7-col>
            </f7-row>
        </f7-block>

        <f7-block class="login-content">
            <f7-card class="login-form-card">
                <f7-row no-gap>
                    <f7-col>
                        <form class="form-store-data" id="demo-form">
                            <f7-list no-hairlines-md>
                                <div class="no-hairlines-md inline-labels">
                                    <ul>
                                        <!-- Campo de URL -->
                                        <li>
                                            <span class="item-title item-label custom-w20 custom-label-top-input">URL</span>
                                            <div class="item-content item-input">
                                                <div class="item-inner">
                                                    <div class="item-title item-label custom-w20">
                                                        <select v-model="internet_protocol">
                                                            <option value="https://">https://</option>
                                                            <option value="http://">http://</option>
                                                        </select>
                                                    </div>
                                                    <div class="item-input-wrap">
                                                        <input type="text" placeholder="demo.facturalo.pro" required validate v-model="url">
                                                        <span class="input-clear-button"></span>
                                                    </div>
                                                </div>
                                            </div>
                                        </li>
                                    </ul>
                                </div>

                                <!-- Campo de Correo Electrónico -->
                                <f7-list-input
                                    floating-label
                                    type="text"
                                    clear-button
                                    label="Correo electrónico"
                                    placeholder="Correo electrónico"
                                    name="name"
                                    required
                                    validate
                                    :value="email"
                                    @input="email = $event.target.value">
                                </f7-list-input>

                                <div class="no-hairlines-md inline-labels">
                                    <ul>
                                        <!-- Campo de Contraseña -->
                                        <li>
                                            <span class="item-title item-label custom-w20 custom-label-top-input-pw">Contraseña</span>
                                            <div class="item-content item-input" style="margin-top:-12px;">
                                                <div class="item-inner">
                                                    <div class="item-input-wrap">
                                                        <input :type="is_type_password ? 'password':'text'" placeholder="Contraseña" required validate v-model="password">
                                                    </div>
                                                    <div class="item-title item-label custom-w10">
                                                        <i class="fas" :class="icon_class_eye" @click="clickShowPassword"></i>
                                                    </div>
                                                </div>
                                            </div>
                                        </li>
                                    </ul>
                                </div>
                            </f7-list>
                        </form>
                    </f7-col>
                </f7-row>
            </f7-card>
        </f7-block>

        <!-- Botón de Iniciar Sesión -->
        <f7-block class="boton-sesion justify-content-center">
            <f7-row tag="p">
                <f7-col>
                    <f7-button class="sign-in-button" @click="login" large fill round>Iniciar sesión</f7-button>
                </f7-col>
            </f7-row>
        </f7-block>
    </f7-page>
</template>

<script>
    import {auth} from "mixins_/auth";
    import logoLight from "assets/images/logo/logo-light.svg";
    import {general_functions} from "mixins_/general_functions"
    import Logo from "components/layout/Logo.vue"
    import {mapState, mapActions} from 'vuex'

    
    export default {
        mixins: [auth, general_functions],
        components: { Logo },
        data: function () {
            return {
                email: "empresa1@gmail.com",
                password: "empresa1@gmail.com",
                url: "empresa1.demo3pro6.ibu.pe",
                logo: '',
                internet_protocol: 'https://',
                is_type_password: true,
                theme: {},
                icon_class_eye: 'fa-eye-slash'
            };
        },
        created() {
            if (localStorage.url_logo) {
                this.logo = localStorage.url_logo
            } else {
                this.logo = logoLight
            }

            this.setStorageApiUrl()
            this.getInitialSettings()
        },
        computed: {
            ...mapState(['userData']),
        },
        methods: {
            ...mapActions(['userLogged']),
            clickShowPassword(){
                this.is_type_password = !this.is_type_password
                if(this.is_type_password){
                    this.icon_class_eye = 'fa-eye-slash'
                } else {
                    this.icon_class_eye = 'fa-eye'
                }
            },
            setStorageApiUrl(){

                const storage_api_url = localStorage.api_url

                if (storage_api_url)
                {
                    if(storage_api_url.includes(this.internet_protocol) || storage_api_url.includes('http://'))
                    {
                        const parse_url = storage_api_url.split('//')
                        if(parse_url.length == 2) this.url = parse_url[1]
                    }
                    else
                    {
                        this.url = storage_api_url
                    }
                }

            },
            saveToken(token, name, email, ruc = null, logo = null, app_logo_base64 = null) {
                localStorage.api_token = token;
                localStorage.user_name = name;
                localStorage.user_email = email;
                localStorage.app_logo_base64 = app_logo_base64;

                localStorage.api_url = `${this.internet_protocol}${this.url}`

                if (ruc && logo)
                {
                    localStorage.ruc = ruc
                    const url_logo = `${this.internet_protocol}${this.url}/storage/uploads/logos/${logo}`

                    localStorage.url_logo = url_logo
                    this.logo = url_logo
                }
                else
                {
                    if(!logo)
                    {
                        this.removeStorage('url_logo')
                        this.logo = null
                    }
                }

            },
            login() {

                if (!this.email || !this.password || !this.url) {
                    return this.showAlert('Los campos son obligatorios.');
                }

                const self = this;

                self.$f7.preloader.show();

                this.$http
                    .post(`${this.internet_protocol}${this.url}/api/login`, {
                        email: this.email,
                        password: this.password
                    })
                    .then(response => {
                        let data = response.data;

                        if (data.success)
                        {
                            self.saveToken(data.token, data.name, data.email, data.ruc, data.app_logo, data.app_logo_base64)
                            this.userLogged({username: data.name, email: data.email})
                            self.setStorage('company', data.company, true)
                            if (data.app_configuration) self.setStorage('app_configuration', data.app_configuration, true)
                            self.$f7router.navigate("/")
                        }
                        else
                        {
                            alert("Usuario o Password incorrectos.");
                        }
                    })
                    .catch(err => {
                        this.showAlert("No se logro conexión con la URL, verifique la URL (Formato url: cliente.dominio.com).");
                    })
                    .then(() => {
                        self.$f7.preloader.hide();
                    });
            },
            getInitialSettings() {
                this.theme = this.getThemeSettings()
            },
        }
    };
</script>

<style scoped>
/* Estilos personalizados para replicar el diseño */
.login-background {
    background-image: url('./bg.jpg');
    background-size: cover;
    background-position: center;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: -1;
}

.login-content {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 80vh;
}

.login-form-card {
    background-color: rgba(255, 255, 255, 0.9);
    border-radius: 15px;
    padding: 20px;
}

.sign-in-button {
    background-color: #ff7f50;
    border-radius: 30px;
    color: white;
}


.item-input input {
    border-radius: 30px;
    background-color: rgba(255, 255, 255, 0.6);
    padding: 10px;
    border: 1px solid rgba(255, 255, 255, 0.3);
}

</style>































