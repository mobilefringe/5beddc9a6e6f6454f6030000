<template>
	<div class="events_container" v-if="dataLoaded"> <!-- template will not render without an outer container -->
	    <div class="inside_page_banner margin_30" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }"></div>
		
	    <div class="promo_dets" v-for="item in dineList">
			<div class="row page_container">
				<div class="col-sm-7" >
					<!--<div class="promo_div_image" v-lazy-container="{ selector: 'img' }">-->
					<!--	<img :data-src="item.image_url"  data-loading='//codecloud.cdn.speedyrails.net/sites/5b16b9686e6f6426c91e0000/image/png/1521035009104/Screen Shot 2018-03-14 at 9.43.24 AM.png' alt=""/>-->
					<!--</div>-->
				</div>
				<div class="col-sm-5 promo_div_dets text-left">
					<p class="promo_div_name">{{ item.name }}</p>
					<p class="promo_div_description">{{ item.description_short }}</p>
					<div class="promo_feature_share row is-table-row">
    					<span class="feature_read_more col-sm-8">
    						<router-link :to="'/stores/'+ item.slug" class="mobile_readmore">
    							<p class="feature-readmore hvr-sweep-to-right" :aria="item.name">
    							    View Details<i class="fa fa-chevron-right pull-right" aria-hidden="true"></i>
							    </p>
    						</router-link>
    					</span>
    					<div class="text-right  col-sm-4" v-if="item">
        					<social-sharing :url="shareURL(item.slug)" :title="item.title" :description="item.body" :quote="_.truncate(item.description, {'length': 99})" twitter-user="" :media="item.image_url" inline-template >
                                <div class="blog-social-share">
                                    <div class="social_share">
                                        <network network="facebook">
                                            <i class="fa fa-facebook"></i>
                                        </network>
                                        <network network="twitter">
                                            <i class="fa fa-twitter"></i>
                                        </network>
                                    </div>
                                </div>
                            </social-sharing>
    					</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style>
    .events_container .promo_container{
        border-top: 1px solid #aea99e;
    }
    .events_container .all_dates span.active { 
        background-color: #bababa;
    }
    .events_container .promo_dets {
        border-bottom: 1px solid #aea99e;
    }
    .events_container .promo_dets:last-child {
        border-bottom: none;
    }
    .events_container .feature_read_more {
        width : auto;
    }
</style>

<script>
    define(["Vue", "vuex", "moment", "moment-timezone", "vue-moment", "vue-meta", "vue-lazy-load"], function(Vue, Vuex, moment, tz, VueMoment, Meta, VueLazyload) {
        Vue.use(Meta);
        Vue.use(VueLazyload);
        return Vue.component("dine-component", {
            template: template, // the variable template will be injected
            data () {
                return {
                    dataLoaded: false,
                     pageBanner: null,
                }  
            },
            created () {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Dine Image');
                    if (temp_repo && temp_repo.images) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    } else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b88438d6e6f641e8d3c0000/image/png/1531495616000/inside_banner.png"
                        }
                    }

                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findRepoByName',
                    'storesByCategoryName'
                ]),
                dineList(){
        
                    var dine_stores = this.storesByCategoryName["Dine"];
                    console.log("dine_stores", dine_stores)
                    _.forEach(dine_stores, function(value, key) {

                        if (_.includes(o.image_url, 'missing')) {
                            o.image_url = "http://placehold.it/930x440";
                        }
                        
                        if (value.description.length > 120) {
                            value.description_short =  _.truncate(value.description, { 'length': 120, 'separator': '...' });
                        } else {
                             value.description_short = value.description;
                        }
         
  
                    });
                    return dine_stores
                    
                }
            },
            methods: {
                loadData: async function() {
                    try {
                        let results = await Promise.all([
                            this.$store.dispatch("getData", "repos"),
                            this.$store.dispatch("getData", "categories")
                        ]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                shareURL(slug){
                    var share_url = "http://rollinghills.ca/events/" + slug;
                    return share_url;
                }
            }
        });
    });
</script>