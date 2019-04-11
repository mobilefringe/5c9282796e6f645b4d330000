<template>
	<div class="event_dets_container" v-if="currentPromo">
		<div class="page_header" v-if="pageBanner" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
			<div class="site_container">
				<div class="header_content caps">
					<h1>{{$t("promos_page.promotions")}}</h1>
					<h2 style="display:none;">Scroll to view event details</h2>
				</div>
			</div>
		</div>
		<div class="site_container">
		    <sponsorship></sponsorship>
			<div class="row">
				<div class="col-sm-12 promo_image_container text-left">
					<router-link to="/promotions">
					    <i class="fa fa-angle-left"></i> 
					    {{$t("events_page.back_to_events")}}
					</router-link>
					<h3 class="promo_name" style="margin: 20px auto 0px;"  v-if="locale=='en-ca'">{{currentEvent.name}}</h3>
					<h3 class="promo_name" style="margin: 20px auto 0px;"  v-else>{{currentEvent.name_2}}</h3>
					<div class="row">
						<p class="promo_div_date pull-left">{{currentEvent.start_date | moment("MMM D", timezone)}} - {{currentEvent.end_date | moment("MMM D", timezone)}}</p>
						<social-sharing :url="$root.shareURL('events',currentPromo.slug)" :title="currentEvent.title" :description="currentEvent.body" :quote="_.truncate(currentEvent.description, {'length': 99})" :twitter-user="$root.twitter_user" :media="currentEvent.image_url" inline-template >
							<div class="blog-social-share pull-right" style="margin: 15px auto;">
								<div class="social_share">
									<network network="facebook">
										<i class="fa fa-facebook social_icons" aria-hidden="true"></i>
									</network>
									<network network="twitter">
										<i class="fa fa-twitter social_icons" aria-hidden="true"></i>
									</network>
								</div>
							</div>
						</social-sharing>
					</div>
					<div class="col-sm-12 no_padding text-left">
						<img v-if="!_.includes(currentEvent.image_url, 'missing')" v-lazy="currentEvent.image_url" class="image" :alt="currentEvent.name"/>
						<div class="text-left promo_description">
							<p v-if="locale=='en-ca'" v-html="currentEvent.rich_description"></p>
							<p v-else v-html="currentEvent.rich_description_2"></p>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
    define(['Vue', 'vuex', 'moment', 'vue-lazy-load', "vue!sponsorship"], function(Vue, Vuex, moment, VueLazyload, sponsorship) {
        Vue.use(VueLazyload);
        return Vue.component("promo-details-component", {
            template: template, // the variable template will be injected,
            props:['id', 'locale'],
            data: function() {
                return {
                    currentPromo: null,
                    storePromos : null,
                    storeHours : null,
                    pageBanner : null
                }
            },
            beforeRouteUpdate(to, from, next) {
                this.currentPromo = this.findPromoBySlug(to.params.id);
                    if (this.currentPromo === null || this.currentPromo === undefined){
                        this.$router.replace('/');
                    }
                next();
            },
            created(){
                this.loadData().then(response => {
                    this.updateCurrentPromo(this.id);
                    var temp_repo = this.findRepoByName('Promos Banner');
                    if(temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    else {
                        this.pageBanner = {};
                        this.pageBanner.image_url = "";
                    }
                    this.promos = this.promotions;
                });
            },
            watch: {
                currentPromo : function (){
                    if(this.currentPromo != null) {
                        if (this.currentPromo.store  && _.includes(this.currentPromo.store.store_front_url_abs, 'missing')) {
                            // this.currentPromo.store.store_front_url_abs = this.property.default_logo_url;
                            this.currentPromo.store.no_store_logo = true;
                        }
                        else if (!this.currentPromo.store) {
                            this.currentPromo.store = {};
                            this.currentPromo.store.store_front_url_abs = this.property.default_logo_url;
                        }
                        // if ( _.includes(this.currentStore.store_front_url_abs, 'missing')) {
                        //   this.currentStore.no_store_logo = true;
                        // } else {
                        //   this.currentStore.no_store_logo = false;
                        // }
                        var vm = this;
                        var temp_promo = [];
                        var current_id =_.toNumber(this.currentPromo.id);
                        _.forEach(this.currentPromo.store.promotions, function(value, key) {
                            if(_.toNumber(value) != current_id){
                                var current_promo = vm.findPromoById(value);
                                current_promo.description_short = _.truncate(current_promo.description, {'length': 70});
                                temp_promo.push(current_promo);
                            }
                        });
                        this.storePromos = temp_promo;
                    }
                    if(this.currentPromo.store) {
                        var storeHours = [];
                        var vm = this;
                        _.forEach(this.currentPromo.store.store_hours, function (value, key) {
                            var hour = vm.findHourById(value);
                            if(hour.day_of_week === 0){
                                hour.order = 7;
                            }
                            else {
                                hour.order = hour.day_of_week;
                            }
                            storeHours.push();
                        });
                        this.storeHours = _.sortBy(storeHours, [function(o) { return o.order; }]);
                    }
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'processedPromos',
                    'findPromoBySlug',
                    'findPromoById',
                    'timezone',
                    'findRepoByName',
                    'findHourById'
                ]),
                allPromos() {
                    return this.processedPromos;
                },
            },
            methods: {
                updateCurrentPromo (id) {
                    this.currentPromo = this.findPromoBySlug(id);
                    if (this.currentPromo === null || this.currentPromo === undefined){
                        this.$router.replace('/');
                    }
                },
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData", "promotions"), this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                }
            }
        });
    });
</script>