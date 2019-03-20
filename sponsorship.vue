<template>
    <div>Helloefwefwe</div>
</template>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta"], function(Vue, Vuex, moment, tz, VueMoment, Meta) {
        Vue.use(Meta);
        return Vue.component("sponsorship", {
            template: template, // the variable template will be injected
            props:['locale'],
            data: function() {
                return {
                    pageBanner : null
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Hours Banner');
                    if(temp_repo && temp_repo.images) {
                        this.pageBanner = temp_repo.images[0];
                    } else {
                        this.pageBanner= {};
                        this.pageBanner.image_url = "";
                    }
                });
                
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'getPropertyHours',
                    'getPropertyHolidayHours',
                    'findRepoByName'
                ]),
                hours () {
                   var hours = _.sortBy(this.getPropertyHours, ['day_of_week']);
                    var ordered_hours = [];
                    _.forEach(hours, function (val, key) {
                       if(val.day_of_week !== 0) {
                           ordered_hours.push(val);
                       }
                    });
                    ordered_hours.push(hours[0]);
                    return ordered_hours;
                },
                holidayHours () {
                    return this.getPropertyHolidayHours;
                },
                reducedHolidays () {
                    var holidayHours = this.holidayHours;
                    return _.filter(holidayHours, function(o) { return !o.is_closed; });
                },
                closeHolidays () {
                    var holidayHours = this.holidayHours;
                    return _.sortBy(_.filter(holidayHours, function(o) { return o.is_closed; }), [function(o) { return o.holiday_date; }]);
                }
            },
            methods : {
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData", "repos")]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
            }
        });
    });
</script>