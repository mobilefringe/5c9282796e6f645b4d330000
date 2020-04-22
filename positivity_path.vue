<template>
    <div id="contact_us_container"> <!-- for some reason if you do not put an outer container div this component template will not render -->
        <div v-if="pageBanner" class="page_header" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content">
					<h1>Positivity Path</h1>
					<h2 style="display:none;">Scroll to  view contact details</h2>
					<h3 style="display:none;">Scroll to  view contact details</h3>
				</div>
			</div>
		</div>  
        <div class="margin_25_across padding_top_40 site_container">
    		<sponsorship></sponsorship>
            <div class="row"> 
                <div class="col-sm-6 text-left" v-if="currentPage && currentPage.body">
                    <div class="text-left contact_us_body" v-html="currentPage.body"></div>
                </div> 
                <div class="col-sm-6 contact_contents">
                    <form class="form-horizontal" action="form-submit" @submit.prevent="validateBeforeSubmit">
                        <div class="form-group ">
                            <div class="col-sm-12" :class="{'has-error': errors.has('name')}">
                                <label class="label" for="contact_name">Name</label>
                                <input v-model="form_data.name" v-validate="'required|alpha_spaces'" class="form-control" :class="{'input': true}" name="name" type="text" placeholder="Name" data-vv-delay="500" id="contact_name">
                                <span v-show="errors.has('name')" class="form-control-feedback">{{ errors.first('name') }}</span>
                            </div>
                            <div class="col-sm-12" :class="{'has-error': errors.has('phone')}">
                                <label class="label" for="contact_phone">Phone Number</label>
                                <input v-model="form_data.phone" v-validate="'required|alpha_spaces'" class="form-control" :class="{'input': true}" name="phone number" type="text" placeholder="Phone Number" data-vv-delay="500" id="contact_phone">
                                <span v-show="errors.has('phone')" class="form-control-feedback">{{ errors.first('phone') }}</span>
                            </div>
                            <div class="col-sm-12" :class="{'has-error': errors.has('email')}">
                                <label class="label" for="contact_email">Email</label>
                                <input v-model="form_data.email" v-validate="'required|email'" class="form-control" :class="{'input': true}" name="email" type="email" placeholder="Email" data-vv-delay="500" id="contact_email">
                                <span v-show="errors.has('email')" class="form-control-feedback">{{ errors.first('email') }}</span>
                            </div>
                            <div class="col-sm-12" :class="{'has-error': errors.has('message')}">
                                <label class="label" for="contact_message">My words of wisdom, sincere statement, or positive phrase</label>
                                <input v-model="form_data.message" v-validate="'required|alpha_spaces'" class="form-control" :class="{'input': true}" name="message" type="text" placeholder="Max. 50 characters" data-vv-delay="500" id="contact_message" maxlength="50">
                                <span v-show="errors.has('message')" class="form-control-feedback">{{ errors.first('message') }}</span>
                            </div>
                            <div class="col-sm-12">
    					        <label class="label">
                                    <input name="agree_name" type="checkbox">
                                    Yes, I would like my First Initial and Last Name included on the Positivity Path Decal!
                                </label>
    					    </div>
    					    <div class="col-sm-12" :class="{'has-error': errors.has('agree')}">
    					        <label class="label">
                                    <input name="agree_rules" required type="checkbox">
                                    Yes, I have read and agree to the <a href="/pages/shoppersmall-positivity-path-rules-regulations" target="_blank">Promotion Rules & Regulations</a>.
                                </label>
                                <span v-show="errors.has('agree')" class="form-control-feedback">{{ errors.first('agree') }}</span>
    					    </div>
                        </div>
                        <div class="form-group account-btn text-left m-t-10">
                            <div class="col-xs-12">
                                <button class="contact_btn" type="submit" :disabled="formSuccess">Submit</button>
                            </div>
                        </div>
                    </form>
                    
                    <div id="send_contact_success" class="alert alert-success text-left" role="alert" v-show="formSuccess">
                        <span class="glyphicon glyphicon-ok" aria-hidden="true"></span>
                        <span class="sr-only">Success</span>
                        Thank you for your submission! A member from our team will contact you shortly.
                    </div>
                    <div id="send_contact_error" class="alert alert-danger text-left" role="alert" v-show="formError">
                        <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
                        <span class="sr-only">Error:</span>
                        There was an error when trying to submit your request. Please try again later.
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    define(["Vue", "vuex", "vue-meta", 'vee-validate', 'utility', "vue!sponsorship"], function(Vue, Vuex, Meta, VeeValidate, Utility, sponsorship) {
        Vue.use(Meta);
        Vue.use(VeeValidate);
        return Vue.component("contact-us-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    form_data : {},
                    formSuccess : false,
                    formError: false,
                    validaNum: '',
                    correctValNum: null,
                    validNumError: false,
                    currentPage : null,
                    pageBanner: null,
                    plainBanner: null
                }
            },
            created(){
                this.loadData().then(response => {
                    this.currentPage = response[0].data;
                    var temp_repo = this.findRepoByName('Contact Us Banner');
                    if (temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    } else {
                        this.pageBanner= {};
                        this.pageBanner.image_url = "";
                    }
                });
                this.$store.dispatch('LOAD_PAGE_DATA', {url: this.property.mm_host + "/pages/" + this.$root.subdomain + "-positivity-path.json"}).then(response => {
                    this.currentPage = response.data;
                });
            },
            mounted () {
                // Ensure the variables are created in this order for email
                this.form_data.name = null;
                this.form_data.phone = null;
                this.form_data.email = null;
                this.form_data.subject = this.property.name + ' Contact Us Form';
                this.form_data.message = null;
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findRepoByName'
                ]),
            },
            methods: {
                validateBeforeSubmit() {
                    this.$validator.validateAll().then((result) => {
                        let errors = this.errors;
                        send_data = {};
                        send_data.append("mailto", "caitlin@mobilefringe.com");
                        send_data.append("from_email", this.form_data.email);
                        send_data.append("subject", this.property.name + " Positivity Path");
                        send_data.append("custom[Name]", this.form_data.name);
                        send_data.append("custom[Email]", this.form_data.email);
                        send_data.append("custom[Phone]", this.form_data.phone);
                        send_data.append("custom[Words of Wisdom]", this.form_data.message);
                        
                        console.log("this.form_data", this.form_data)
                        // send_data.form_data = JSON.stringify(Utility.serializeObject(this.form_data));
                        // this.$store.dispatch("CONTACT_US", send_data).then(res => {
                        //     this.formSuccess = true;
                        // }).catch(error => {
                        //     try {
                        //         if (error.response.status == 401) {
                        //             console.log("Data load error: " + error.message);
                        //             this.formError = true;
                        //         } 
                        //         else {
                        //             console.log("Data load error: " + error.message);
                        //             this.formError = true;
                        //         }
                        //     } 
                        //     catch (e) {
                        //         console.log("Data load error: " + error.message);
                        //         this.formError = true;
                        //     }
                        // })
                    })
                },
                loadData: async function() {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "repos")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                }
            }
        });
    });
</script>
