<template>
	<div class="page_container promotions_container" v-if="dataloaded"><!-- for some reason if you do not put an outer container div this component template will not render -->
		<div class="date_bar">
		    <span v-on:click="beforeDate()"> <i class="fa fa-chevron-left"></i></span>
		    <span class="current_date">
		        {{selectedDate}}
		    </span>
		    <span v-on:click="afterDate()"> <i class="fa fa-chevron-right"></i></span>
		</div>
		<div class="all_dates text-center">
		    <span v-for="n in daysInMonth" :id="'date_'+n.date" v-on:click="newDate(n.date)" :class="{active: n.isActive}">{{n.date}}</span>
		</div>
		<div class="promo_container" v-if="filteredPromos.length > 0">
			<div class="row promo_dets is-table-row" v-for="promo in filteredPromos">
				<div class="col-sm-7" >
					<div class="promo_div_image">
						<!--<img :src="promo.store.image_url"  :alt="promo.store.name"/>-->
						<img :src="promo.image_url"  :alt="promo.name"/>
					</div>
				</div>
				<div class="col-sm-5 promo_div_dets text-left">
					<p class="promo_div_name">{{promo.name}}</p>
					<p class="promo_div_store_name" v-if="promo.store"><router-link :to="'/stores/'+promo.store.slug">{{promo.store.name | uppercase}}</router-link></p>
					<p class="promo_div_date">{{promo.start_date | moment("MMM D", timezone)}} - {{promo.end_date | moment("MMM D", timezone)}}</p>
					<p class="promo_div_description">{{promo.description_short}}</p>
					<div class="promo_feature_share row is-table-row">
    					<span class="feature_read_more col-sm-8">
    						<router-link :to="'/promotions_and_events/'+promo.slug" class="mobile_readmore" >
    							<p class="feature-readmore  hvr-sweep-to-right" :aria="promo.name"><span v-if="promo.type == 'event'">View Event Details</span><span v-else>View Promotion Details</span> <i class="fa fa-chevron-right pull-right" aria-hidden="true"></i></p>
    						</router-link>
    					</span>
    					<div class="text-right  col-sm-4" v-if="promo">
        					<social-sharing :url="shareURL(promo.slug)" :title="promo.title" :description="promo.body" :quote="_.truncate(promo.description, {'length': 99})" twitter-user="" :media="promo.image_url" inline-template >
                                <div class="blog-social-share">
                                    <div class="social_share">
                                        <network network="facebook">
                                        <img src="//codecloud.cdn.speedyrails.net/sites/5b16b9686e6f6426c91e0000/image/png/1511797683914/facebook.png" class="social_icons" alt="">
                                    </network>
                                    <network network="twitter">
                                        <img src="//codecloud.cdn.speedyrails.net/sites/5b16b9686e6f6426c91e0000/image/png/1511797704155/twitter.png" class="social_icons" alt="">
                                    </network>
                                    </div>
                                </div>
                            </social-sharing>
    					</div>
    					
					</div>
				</div>
			</div>
		</div>
		<div class="promo_container" v-else>
		    <p style="padding:20px"> Sorry, No promotions or events available for this day. </p>
		</div>
	</div>
</template>

<style>
    .date_bar{
        /* Today: */
        background: #D3D3D3;
        height: 40px;
        line-height: 40px;
        margin: auto;
        text-align: center;
    }
    .date_bar .fa{
        cursor: pointer;
    }
    .current_date{
        color: #4a4a4a;
        padding: 0 10px;
    }
    .all_dates {
        border-bottom: 1px solid #aea99e;
        display: flex;
        flex-wrap: wrap;
    }
    .all_dates span {
        font-size: 16px;
        color: #000000;
        letter-spacing: 1.5px;
        height: 40px;
        line-height: 40px;
        padding: 5px;
        cursor: pointer;
    }
     .all_dates [class*="date_"]:focus, [class*="date_"]:hover { 
        background-color: #D3D3D3;
    }
    .all_dates span.active { 
        background-color: #bababa;
    }
    .promo_dets {
        border-bottom: 1px solid #aea99e;
    }
    .row.is-table-row {
        margin: 0;
    }
    .row.is-table-row [class*="col-"] {
        padding:0;
    }
    .feature_read_more {
        width : auto;
    }
    
</style>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", "vue-lazy-load"], function(Vue, Vuex, moment, tz, VueMoment, Meta, VueLazyload) {
        Vue.use(Meta);
        Vue.use(VueLazyload);
        return Vue.component("promos-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    selectedDate: null,
                    filteredPromos:[],
                    dataloaded: false
                }
            },
            created() {
                this.loadData().then(response => {
                    this.dataloaded = true;
                    this.selectedDate = moment().tz(this.timezone).format('MMM D, YYYY');
                    var date = moment(this.selectedDate).date();
                    this.daysInMonth[date-1].isActive = true;
                });
                // this.$store.dispatch("getData", "promotions").then(response => {
                    
                // }, error => {
                //   console.error("Could not retrieve data from server. Please check internet connection and try again.");
                // });
            },
            watch: {
                selectedDate: function() {
                    //sort promos
                    selected = moment(this.selectedDate).format('MM DD YYYY');
                    var vm = this;
                    vm.filteredPromos = _.filter(vm.promotions, function(val){
                        start_date = moment(val.start_date).tz(vm.timezone).format('MM DD YYYY');
                        end_date = moment(val.end_date).tz(vm.timezone).format('MM DD YYYY');
                        return (selected <= end_date && selected >= start_date && selected <= end_date);
                    });
                    //remove old active class, change active class 
                    old_date = moment(this.selectedDate).date();
                    this.daysInMonth.map(date => {
                        date.isActive = false;
                    });
                    this.daysInMonth[old_date-1].isActive = true;
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'processedPromos',
                    'processedEvents'
                ]),
                promotions() {
                    var promos_events =this.processedPromos;
                    var vm = this;
                    var temp_promo = [];
                    var temp_job = [];
                    _.forEach(promos_events, function(value, key) {
                        var today = moment.tz(this.timezone).format();
                        var showOnWebDate = moment.tz(value.show_on_web_date, this.timezone).format();
                        if (today >= showOnWebDate) {
                            value.description_short = _.truncate(value.description, {
                                'length': 70
                            });
                            
                            // if (value.store != null && value.store != undefined && _.includes(value.store.image_url, 'missing')) {
                            //     value.store.image_url = vm.property.default_logo_url;
                            // }
                            // else if ( value.store == null ||  value.store == undefined) {
                            //     value.store = {};
                            //     value.store.image_url =  vm.property.default_logo_url;
                            // }
                            
                            if (_.includes(value.image_url, 'missing')) {
                                if (value.store != null && value.store != undefined) {
                                    if(_.includes(value.store.store_front_url_abs, 'missing')) {
                                        value.image_url = vm.property.default_logo_url;    
                                    } else {
                                        value.image_url = value.store.store_front_url_abs;
                                    }
                                } else {
                                    value.image_url = vm.property.default_logo_url; 
                                }
                            } else {
                                value.image_url = value.image_url;
                            }
                            
                            if(value.eventable_id) {
                                value.type = "event"
                            }
                            else {
                                value.type = "promotion"
                            }
                            temp_promo.push(value);
                        }
                    });
                    
                    return _.sortBy(temp_promo, [function(o) { return o.start_date; }]);
                },
                daysInMonth() {
                    var days = moment(this.selectedDate).daysInMonth();
                    var all_days= [];
                    for(var i = 1; i <= days; i++) {
                        var temp_day = {};
                        temp_day.date = i;
                        temp_day.isActive = false;
                        
                        all_days.push(temp_day);
                    }
                    return all_days;
                }
            },
            methods: {
                loadData: async function() {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "promotions"), this.$store.dispatch("getData", "events")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                beforeDate: function(ev) {
                    selectedDate = this.selectedDate;
                    selectedDate = moment(selectedDate).subtract(1, 'days');
                    this.selectedDate = moment(selectedDate).tz(this.timezone).format('MMM D, YYYY');
                },
                afterDate() {
                    selectedDate = this.selectedDate;
                    selectedDate = moment(selectedDate).add(1, 'days');
                    this.selectedDate = moment(selectedDate).tz(this.timezone).format('MMM D, YYYY');
                },
                newDate (val) {
                    old_date = moment(this.selectedDate).date();
                    month = moment(this.selectedDate).month();
                    year = moment(this.selectedDate).year();
                    this.selectedDate = moment([year, month, val]).format('MMM D, YYYY');
                },
                shareURL(slug){
                    var share_url= "https://holtrenfrewcentre.ca/promotions_and_events/" + slug;
                    // if(promo.type = "event") {
                        
                    // } else {
                    //     share_url= "http://holtrenfrewcentre.ca/promotions/" + slug;
                    // }
                    return share_url;
                },
            }
        });
    });
</script>