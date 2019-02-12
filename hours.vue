<template>
    <div class="hours_page_container"> <!-- for some reason if you do not put an outer container div this component template will not render -->
        <img style= "width: 100%;" :src="hoursSideImage.image_url" alt="" class="show_phone">  
        <div class="page_container margin_30">
            <div class="all_hours_container">
                <div class="row">
                    <div class="col-md-6">
                        <h2 class="hours_heading text-left" >Regular Shopping Hours</h2>
                        <div id="hours_container" class="hours_container">
                            <div class="hours_div text-left" v-for="hour in hours">
                                <span>{{hour.day_of_week | moment("dddd", timezone)}}:</span>
                                <span>{{hour.open_time | moment("h:mm A", timezone)}} - {{hour.close_time | moment("h:mm A", timezone)}}</span>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <h2 class="hours_heading text-left">Holiday Hours</h2>
                        <div id="holidays_hours_container" class="hours_container">
                            <div class="hours_div text-left"  v-for="hour in reducedHolidays">
                                <span>{{hour.holiday_name}} ({{hour.holiday_date | moment("MMM D YYYY", timezone)}})</span>
                                <span>{{hour.open_time | moment("h:mm A", timezone)}} - {{hour.close_time | moment("h:mm A", timezone)}}</span>
                            </div>
                        </div>
                        <div class="margin_30"></div>
                        <h2 class="hours_heading text-left">Holiday Closures</h2>
                        <div id="closed_hours_container" class="hours_container">
                            <div class="hours_div text-left" v-for="hour in closeHolidays">
                                <span>{{hour.holiday_name}} ({{hour.holiday_date | moment("MMM D YYYY", timezone)}})</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d3315.8160866223798!2d-118.33180808482304!3d33.79124718067779!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x80dd4a44b52715a9%3A0xd7414911d196d0b7!2sRolling+Hills+Plaza!5e0!3m2!1sen!2sca!4v1549987085186" width="100%" height="450" frameborder="0" style="border:0" allowfullscreen></iframe>
    </div>
</template>
<style>
    .hours_page_container .row{
        margin-left:inherit;
    }
</style>
<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment"], function(Vue, Vuex, moment, tz, VueMoment) {
        return Vue.component("hours-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    hoursSideImage:null
                }
            },
            created() {
                this.loadData().then(response => {
                    this.dataloaded = true;
                    
                    var temp_repo = this.findRepoByName('Hours Side Image');
                    if(temp_repo) {
                        this.hoursSideImage = temp_repo.images[0];
                    } else {
                        this.hoursSideImage = {};
                        this.hoursSideImage.image_url = "";
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
                    return this.getPropertyHours;
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
            methods: {
                loadData: async function() {
                    try {
                        let results = await Promise.all([ this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
            }
        });
    });
</script>
