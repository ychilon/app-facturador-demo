<template>
<f7-page :class="theme.class_bg_body">

    <!-- logo de cambio de pagina -->
    <f7-popup class="demo-popup" :opened="splash" @popup:closed="popupOpened = false">
        <f7-page :class="theme.class_bg_header">
            <f7-block class="padding-vertical no-margin-vertical" :class="theme.class_bg_body">
                <br><br><br>
            </f7-block>
            <f7-block class="padding-vertical display-flex justify-content-center no-margin-vertical" v-if="logo" :class="theme.class_bg_body">
                <logo heightLogo="80" alternativePercentage="40%" :isDark="true" class="display-flex justify-content-center"></logo>
                <!-- <img :width="width_img" :height="height_img" class="center padding-vertical margin-vertical margin-horizontal" :src="logo" alt /> -->
            </f7-block>
            <f7-block class="display-flex justify-content-center no-margin" :class="theme.class_bg_body">
                <img :src="img_icons" alt="icons" width="70%" class="center">
            </f7-block>
            <f7-block :class="[configuration.header_waves === 1 ? 'display-flex' : 'display-none', theme.class_bg_body]" class="no-margin-top no-margin-vertical no-padding-horizontal">
                <div class="waves" style="transform: rotate(0deg);">
                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 100">
                        <path d="M0 0 A100 100 0 0 0 200 0 L200 0 L0 0 Z" fill="#2B614D"/>
                    </svg>
                </div>
            </f7-block>
        </f7-page>
    </f7-popup>

    <header-layout></header-layout>

    <f7-block class="bg-color-red padding" v-if="isOffline">
        <p class="text-color-white text-align-center">SIN ACCESO A INTERNET, VERIFICA TU CONEXIÓN.</p>
    </f7-block>

    <f7-block v-if="!hasPermissions">
        <f7-card class="card-warning">
            <f7-card-content class="">
                <f7-row>
                    <f7-col width="80">
                        <p class="text-color-white">No tiene permisos asignados, comuníquese con el administrador.</p>
                    </f7-col>
                    <f7-col width="20">
                        <span class="material-iconspadding-top icon-card-warning">warning</span>
                    </f7-col>
                </f7-row>
            </f7-card-content>
        </f7-card>
    </f7-block>

</f7-page>
</template>

<script>

    import {mapActions} from 'vuex'
    import logoOficial from "assets/images/logo/logo-oficial.svg";
    import logoOficialLight from "assets/images/logo/logo-light.svg";
    import icons from "assets/images/icons-01.svg";
    import HeaderLayout from "components/layout/Header";
    import BaseIcon from 'components/layout/BaseIcon.vue';
    import {auth} from "mixins_/auth";
    import {general_functions, set_logo, functions_direct_print} from "mixins_/general_functions"
    import Logo from "components/layout/Logo"

    export default {
        mixins: [auth, general_functions, set_logo, functions_direct_print],
        components: {
            HeaderLayout,
            BaseIcon,
            logoOficial,
            Logo
        },
        data: function () {
            // Must return an object
            return {
                logo: logoOficialLight,
                logo_oficial: '',
                user: "",
                password: "",
                splash: true,
                isOffline: false,
                width_img: '45%',
                height_img: '45%',
                img_icons: icons,
                configuration: {},
                permissions: [],
                is_pos_mode: false,
                theme: {}
            };
        },
        async created() {

            this.userLogged({})

            await this.setInitialLogo()
            await this.getInitialSettings()

            var self = this;
            window.addEventListener("online", function () {
                self.isOffline = false;
            });
            window.addEventListener("offline", function () {
                self.isOffline = true;
            });

            this.checkDirectPrint()

        },
        mounted() {
            setTimeout(this.verifytoken, 1000); // 2500);
        },
        computed:{
            hasPermissions(){
                return this.permissions.length > 0
            }
        },
        methods: {
            ...mapActions(['userLogged']),
            checkDirectPrint()
            {
                if(this.is_pos_mode)
                {
                    const configuration = this.getInitialConfiguration()

                    if(this.isAllowedDirectPrint(configuration))
                    {
                        this.printerDisconnect(configuration.printer_name)
                        this.printerConnect(configuration.printer_name)
                    }
                }
            },
            setInitialLogo(){

                if (localStorage.url_logo)
                {
                    this.logo = localStorage.url_logo
                }

            },
            appendStyleByContent(content){

                const style = document.createElement('style')
                style.textContent = content
                document.head.appendChild(style)

            },
            setStyleThemeContent(style_settings){

                if(style_settings.style_theme_content)
                {
                    switch (style_settings.theme_color)
                    {
                        case 'red':
                        case 'dark':
                            this.appendStyleByContent(style_settings.style_theme_content)
                            break
                    }
                }

            },
            setStyleCardContent(style_settings){

                if(style_settings.style_card_content)
                {
                    switch (style_settings.card_color)
                    {
                        case 'unicolor':
                            this.appendStyleByContent(style_settings.style_card_content)
                            break
                    }
                }

            },
            async setInitialSettings(data){
                await this.setAppMode('pos')
                await this.setPermissions(data)
                await this.setAppLogo(data.generals)
                await this.setGenerals(data)
                await this.setMultiUser(data.multi_user_enabled)

            },
            setAppMode(new_mode){
                this.setStorage('app_mode', new_mode)
                this.is_pos_mode = this.isPosMode(new_mode)
                if(this.is_pos_mode)
                {
                    this.checkCurrentOrientation()
                    // this.redirectRoute('/list-items-sale/')
                }
                this.$eventHub.$emit('appMode', new_mode)
            },
            setMultiUser(multi_user_enabled)
            {
                this.setStorage('multi_user_enabled', multi_user_enabled)
            },
            checkCurrentOrientation()
            {
                const context = this

                if (window.matchMedia('(orientation: portrait)').matches)
                {
                    context.redirectRoute('/list-items-sale/')
                }

                if (window.matchMedia('(orientation: landscape)').matches)
                {
                    context.redirectRoute('/landscape-pos/')
                }
            },
            setGenerals(data){

                this.setStorage('generals', data.generals, true)
                this.$eventHub.$emit('updateGenerals', data.generals)

            },
            setPermissions(data){

                this.permissions = data.permissions
                this.setStorage('permissions', this.permissions, true)
                this.$eventHub.$emit('updatePermissions', this.permissions)

            },
            checkPermissions(value){

                const row = _.find(this.permissions, {value: value})

                return !_.isEmpty(row)

            },
            async getInitialSettings(){

                if(!this.getStorage('api_token')) return

                this.showLoading()
                this.splash = true

                await this.$http.get(`${this.returnBaseUrl()}/app-configurations/initial-settings`,  this.getHeaderConfig())
                                .then(response => {

                                    this.setInitialSettings(response.data)

                                    this.configuration = this.getInitialConfiguration()
                                    this.configuration.header_waves = response.data.style_settings.header_waves
                                    this.setStorage('app_configuration', this.configuration, true)
                                    // this.$f7router.navigate("/"); // no funciona a la primera
                                })
                                .catch(error => {
                                    console.log(error)
                                    this.showAlert(`Ocurrió un error al cargar la configuración inicial: ${error.response.data.message}`)
                                })
                                .finally(() => {
                                    this.hideLoading()
                                    this.splash = false
                                })

                this.theme = this.getThemeSettings()
            },
            go(name) {
                const self = this;

                if (self.isOffline) {
                    return false;
                }

                switch (name) {
                    case "ls_doc":
                        self.$f7router.navigate("/documents/");
                        break;
                    case "nw_doc":
                        self.$f7router.navigate("/form-document/01");
                        break;
                    case "nw_docb":
                        self.$f7router.navigate("/form-document/03");
                        break;
                    case "report-sales":
                        self.$f7router.navigate("/report-sales/");
                        break;
                    case "cpe":
                        self.$f7router.navigate("/cpe/");
                        break;
                    case "sale_note":
                        self.$f7router.navigate("/form-sale-note/");
                        break;
                    case "order_note":
                        self.$f7router.navigate("/form-order-note/");
                        break;
                    case "purchase":
                        self.$f7router.navigate("/form-purchase/");
                        break;
                    case "dispatch":
                        self.$f7router.navigate("/form-dispatch/");
                        break;
                    case "dispatch-carrier":
                        self.$f7router.navigate("/form-dispatch-carrier/");
                        break;
                    case "items":
                        self.$f7router.navigate("/items/")
                        break
                    case "customers":
                        self.$f7router.navigate("/customers/")
                        break
                    case "cash":
                        self.$f7router.navigate("/cash/")
                        break
                    case "quotation":
                        self.$f7router.navigate("/form-quotation/")
                        break
                }
            },
            verifytoken() {
                const self = this;
                if (!localStorage.api_token) {
                    this.splash = false;
                    self.$f7router.navigate("/login/");
                } else {
                    // this.splash = false;
                }
            }
        }
    };
</script>
