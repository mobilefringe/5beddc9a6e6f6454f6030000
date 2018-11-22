<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <!--<div class="inside_header_background" :style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">-->
                <!--    <div class="main_container">-->
                <!--        <div class="page_container">-->
                <!--            <h2>Centre Map</h2>-->
                <!--        </div>-->
                <!--    </div>-->
                <!--</div>-->
                <div class="page_container margin_30">
                    <div class="row">
                        <div class="col_3">
                            <div class="hidden_phone">
                                <h3 class="inside_page_title">Find Store</h3>
                                <div class="store_list_container hidden-mobile" v-if="allStores">
                                    <p class="store_name" v-for="store in allStores" v-on:click="dropPin(store)">{{store.name}}</p>
                                </div>
                            </div>
                            <div class="visible_phone">
                                <v-select 
                                    :options="allStores" 
                                    :placeholder="'Select A Store'" 
                                    :searchable="false" 
                                    :label="'name'" 
                                    :on-change="dropPin"
                                ></v-select>
                            </div>
                        </div>
                        <div class="col-md-9">
                            <mapplic-map ref="mapplic_ref" :height="566" :minimap= "false" :deeplinking="false" :sidebar="false" :hovertip="true" :maxscale= "5" :storelist="allStores" :floorlist="floorList" tooltiplabel="View Store Details"></mapplic-map>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
<script>
    define(["Vue", "vuex", "vue!mapplic-map", "vue-select"], function(Vue, Vuex, MapplicComponent, VueSelect) {
        Vue.component('v-select', VueSelect.VueSelect);
        return Vue.component("stores-component", {
            template: template, // the variable template will be injected
            props:['inside_banner'],
            data: function() {
                return {
                    dataLoaded: false,
                    pageBanner: null
                }
            },
            created (){
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Map Banner');
                    if(temp_repo && temp_repo.images) {
                       temp_repo = temp_repo.images;
                       this.pageBanner = temp_repo[0];
                    }
                    else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5b88438d6e6f641e8d3c0000/image/png/1531495616000/inside_banner.png"
                        }
                    }
                    this.getSVGMap;
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    "property",
                    "timezone",
                    "findRepoByName",
                    "processedStores",
                ]),
                allStores() {
                    var all_stores = this.processedStores;
                    _.forEach(all_stores, function(value, key) {
                        value.zoom = 2;
                    });
                    var initZoom = {};
                    initZoom.svgmap_region = "init";
                    initZoom.z_coordinate = 1;
                    initZoom.x = 0.5;
                    initZoom.y = 0.5;
                    initZoom.zoom = 1;
                    all_stores.push(initZoom)
                    return all_stores
                },
                getSVGMap(){
                  return "//mallmaverick.com"+this.property.svgmap_url;  
                },
                // getSVGMap() {
                //     var svg_maps = this.findRepoByName("SVG Maps")
                //     if(svg_maps != null && svg_maps !== undefined){
                //         svg_maps = svg_maps.images;
                //         var floor_one = "";
                //         var floor_two = "";
                //         _.forEach(svg_maps, function(value, key) {
                //             if(value.id == 41084) {
                //                 floor_one = _.split(value.image_url, '?');
                //                 floor_one = floor_one[0];
                //             }
                //             if (value.id == 41085) {
                //                 floor_two = _.split(value.image_url, '?');
                //                 floor_two = floor_two[0];
                //             }
                //         });
                //         this.floorOne = floor_one;
                //         this.floorTwo = floor_two;
                //     }
                // },
                // getMiniMap () {
                //     var svg_maps = this.findRepoByName("PNG Mini Map").images 
                //     var floor_one = "";
                //     var floor_two = "";
                //     _.forEach(svg_maps, function(value, key) {
                //         if(value.id == 37990) {
                //             floor_one = _.split(value.image_url, '?');
                //             floor_one = floor_one[0];
                //         }
                //         if (value.id == 37991) {
                //             floor_two = _.split(value.image_url, '?');
                //             floor_two = floor_two[0];
                //         }
                //     });
                //     this.miniOne = floor_one;
                //     this.miniTwo = floor_two;
                // },
                floorList () {
                    var floor_list = [];
                    
                    var floor_1 = {};
                    floor_1.id = "first-floor";
                    floor_1.title = "Level One";
                    floor_1.map = this.getSVGMap;
                    // floor_1.minimap = this.miniOne;
                    floor_1.z_index = null;
                    floor_1.show = true;
                    floor_list.push(floor_1);
                    
                    // var floor_2 = {};
                    // floor_2.id = "second-floor";
                    // floor_2.title = "Level Two";
                    // floor_2.map = this.floorTwo;
                    // // floor_2.minimap = this.miniTwo;
                    // floor_2.z_index = 2;
                    // floor_2.show = false;
                    // floor_list.push(floor_2);
                    
                    return floor_list;
                }
            },
            methods: {
                loadData: async function() {
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "repos")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                onOptionSelect(option) {
                    this.$nextTick(function() {
                        this.storeSearch = ""
                    });
                    this.svgMapRef.addMarker(option);
                },
                dropPin(store) {
                    this.$refs.mapplic_ref.showLocation(store.svgmap_region);
                }
            }
        });
    });
</script>
<!--<template>-->
<!--	<div v-if="dataloaded" id="stores_container">-->
<!--		<div class="page_header all_caps double_border_bottom">-->
<!--			<h2 class="page_container text_left page_header_title"> Stores Directory & Map </h2>-->
<!--		</div>-->
<!--		<div class="page_container">-->
<!--			<div class="store-list-left-block col-sm-4 padding_top_20">-->
<!--			    <div style="position:relative; z-index:10;">-->
<!--    			    <p class="text_left">Select Stores: </p>-->
<!--    				<div class="alphabet-dd show_phone" >-->
<!--    				    <v-select v-model="selectedStore" :options="filteredStores" :searchable="false" id="mobile_alpha_list" label="name" placeholder="Select a Store" transition="menu-fade"></v-select>-->
<!--    			    </div>-->
<!--			    </div>-->
<!--		        <div id="mapsvg_store_detail_1" class="show_phone">-->
<!--					<mapplic-map ref="svgmaprefmobile" :height="500" :minimap= "false" :deeplinking="false" :sidebar="false" :hovertip="true" :storelist="mapStores" :floorlist="floorList" :bindLocationOpened="true" :svgWidth="2500" :svgHeight="2500"  v-if="mobile_store"></mapplic-map>-->
<!--				</div>-->
<!--				<div class="store-list-container hidden_phone">-->
<!--					<div class="dropdown_container hidden_phone">-->
<!--						<h3 class="text_left">Select Category</h3>-->
<!--						<div class="category-DD-div">-->
<!--							<v-select v-model="selectedCat" :options="dropDownCats" :searchable="false" :on-change="filterByCategory" transition="menu-fade"></v-select>-->
<!--						</div>-->
<!--						<h3 class="text_left" style=" margin-top: 10px;">Sort Alphabetically</h3>-->
<!--						<div class="category-DD-div">-->
<!--							<v-select v-model="selectedAlpha" :options="alphabet" :searchable="false" :on-change="filterStores" placeholder="All" transition="menu-fade"></v-select>-->
<!--						</div>-->
<!--					</div>-->
					
<!--					<ul class="store-listing text_left  padding_top_20">-->
<!--						<li v-for="store in filteredStores" class="pointer">-->
<!--							<p class="directory_store_name" v-on:click="dropPin(store)" v-if="store.svgmap_region">{{store.name}}</p>-->
<!--							<p class="directory_store_name" v-else><router-link :to="'/stores/'+store.slug">{{store.name}}</router-link></p>-->
<!--						</li>-->
<!--					</ul>-->
<!--				</div>-->
<!--			</div>-->
<!--			<div class="col-sm-8 padding_top_20">-->
<!--				<div id="mapsvg_store_detail" class=" map">-->
<!--					<mapplic-map ref="svgmap_ref" :height="738" :minimap= "false" :deeplinking="false" :sidebar="false" :hovertip="true" :storelist="mapStores" :floorlist="floorList" :svgWidth="2500" :svgHeight="2500" v-if="!mobile_store"></mapplic-map>-->
<!--				</div>-->
<!--			</div>-->
<!--		</div>-->
<!--	</div>-->
<!--</template>-->

<!--<style>-->
<!--    #stores_container .v-select .open-indicator {-->
<!--        right: 0 !important;-->
<!--        top: 0 !important;-->
<!--        bottom: initial !important;-->
<!--        height: 100% !important;-->
<!--        padding: 15px 25px 15px 20px !important;-->
<!--        background-color: #9B9B9B !important;-->
<!--        border-color: #fff !important;-->
<!--        color: #fff !important;-->
<!--    }-->

<!--    #stores_container .v-select .open-indicator:before {-->
<!--        border-color: #fff !important;-->
<!--        margin-top: -5px;-->
<!--    }-->

<!--    #stores_container .v-select .dropdown-menu {-->
<!--        font-family: arial;-->
<!--        font-size: 14px;-->
<!--        color: #000000;-->
<!--        letter-spacing: 1.56px;-->
<!--        display: block;-->
<!--        transition: all 0.3s ease;-->
<!--    }-->

<!--    #stores_container .v-select .dropdown-toggle {-->
<!--        text-align: left;-->
<!--        border-radius: initial;-->
<!--    }-->
<!--    #stores_container .v-select .selected-tag{-->
<!--        position:absolute;-->
<!--        left: 0;-->
<!--    }-->
<!--    #stores_container .v-select li.active.hightlight {-->
<!--        cursor: none!important;-->
<!--    }-->

<!--	#stores_container .text {-->
<!--	    color:white;-->
<!--	    font-size:16px;-->
<!--	    padding-top:2px;-->
<!--	}-->

<!--</style>-->

<!--<script>-->
<!--    define(["Vue", "vuex", "vue-select", "vue!mapplic-svg"], function(Vue, Vuex, VueSelect, MapplicComponent) {-->
<!--        return Vue.component("stores-component", {-->
            template: template, // the variable template will be injected
<!--            data: function() {-->
<!--                return {-->
<!--                    dataloaded: false,-->
<!--                    selectedCat: "All",-->
<!--                    selectedAlpha: null,-->
                    // alphabet: ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"],
<!--                    filteredStores: null,-->
<!--                    selectedStore: null,-->
<!--                    mobile_store: false,-->
<!--                    windowWidth: 0-->
<!--                }-->
<!--            },-->
<!--            created (){-->
<!--                this.loadData().then(response => {-->
<!--                    this.dataloaded = true;-->
<!--                    this.filteredStores = this.allStores;-->
                    
<!--                    this.$on('updateMap', this.updateSVGMap);-->
<!--                });-->
<!--            },-->
<!--            watch: {-->
<!--                windowWidth: function() {-->
<!--                    if (this.windowWidth <= 768) {-->
<!--                        this.mobile_store = true;-->
<!--                    } else {-->
<!--                        this.mobile_store = false;-->
<!--                    }-->
<!--                },-->
<!--                selectedStore () {-->
<!--                    if(this.selectedStore.svgmap_region) {-->
<!--                        this.dropPin(this.selectedStore);-->
<!--                    } else {-->
<!--                        this.$router.push("/stores/"+this.selectedStore.slug);-->
<!--                    }-->
<!--                }-->
<!--            },-->
<!--            mounted() {-->
<!--                this.$nextTick(function() {-->
<!--                    window.addEventListener('resize', this.getWindowWidth);-->
                    //Init
<!--                    this.getWindowWidth();-->
<!--                });-->
<!--            },-->
<!--            methods: {-->
<!--                loadData: async function() {-->
<!--                    try {-->
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
<!--                        let results = await Promise.all([this.$store.dispatch("getData", "categories"), this.$store.dispatch("getData", "repos")]);-->
<!--                    } catch (e) {-->
<!--                        console.log("Error loading data: " + e.message);-->
<!--                    }-->
<!--                },-->
<!--                filterByCategory(category_id) {-->
<!--                    if (category_id == "All" || category_id == null || category_id == undefined) {-->
<!--                        category_id = "All";-->
<!--                    } else {-->
<!--                        category_id = this.findCategoryByName(category_id).id;-->
<!--                    }-->

<!--                    this.breakIntoCol = false;-->
<!--                    if (category_id == "All") {-->
                        this.filteredStores = this.allStores; //this.storesByAlphaIndex;
                        // this.breakIntoCol = true;
<!--                    } else {-->

<!--                        var find = this.findCategoryById;-->
<!--                        var filtered = _.filter(this.allStores, function(o) {-->
<!--                            return _.indexOf(o.categories, _.toNumber(category_id)) > -1;-->
<!--                        });-->
<!--                        this.filteredStores = filtered;-->
<!--                    }-->
<!--                },-->
<!--                updateSVGMap(map) {-->
<!--                    this.map = map;-->
<!--                },-->
<!--                getWindowWidth(event) {-->
<!--                    this.windowWidth = window.innerWidth;-->
<!--                },-->
<!--                dropPin(store) {-->
<!--                     if(this.windowWidth <= 768 && this.selectedStore) {-->
<!--                        this.svgMapRef.showLocation(store.svgmap_region);-->
<!--                    }-->
<!--                    else if(this.windowWidth > 768) {-->
<!--                        this.svgMapRef.showLocation(store.svgmap_region);-->
<!--                    }-->
<!--                },-->
<!--            },-->
<!--            computed: {-->
<!--                ...Vuex.mapGetters([-->
<!--                    'property',-->
<!--                    'timezone',-->
<!--                    'processedStores',-->
<!--                    'processedCategories',-->
<!--                    'storesByAlphaIndex',-->
<!--                    'storesByCategoryName',-->
<!--                    'findCategoryById',-->
<!--                    'findCategoryByName',-->
<!--                    'findRepoByName'-->
<!--                ]),-->
<!--                mapStores() {-->
<!--                    var all_stores = this.processedStores;-->
<!--                    _.forEach(all_stores, function(value, key) {-->
<!--                        value.zoom = 2;-->
<!--                    });-->
                    
<!--                    var initZoom = {};-->
<!--                    initZoom.svgmap_region = "init";-->
<!--                    initZoom.z_coordinate = 1;-->
<!--                    initZoom.x = 0.5;-->
<!--                    initZoom.y = 0.5;-->
<!--                    initZoom.zoom = 1;-->
<!--                    initZoom.name = "init";-->
<!--                    all_stores.push(initZoom)-->
                    
<!--                    return all_stores-->
<!--                },-->
<!--                allStores() {-->
<!--                    var stores = this.processedStores;-->
<!--                    stores.map(function(store){-->
<!--                        store.zoom = 2;-->
<!--                    });-->
<!--                    stores = _.filter(stores, function(o) { return o.name != "init" })-->
<!--                    return stores;-->
<!--                },-->
<!--                allCatergories() {-->
<!--                    return this.processedCategories;-->
<!--                },-->
<!--                dropDownCats() {-->
<!--                    var cats = _.map(this.$store.getters.processedCategories, 'name');-->
<!--                    cats.unshift('All');-->
<!--                    return cats;-->
<!--                },-->
<!--                alphabet() {-->
<!--                    return _.keys(this.storesByAlphaIndex);-->
<!--                },-->
<!--                findCategoryById() {-->
<!--                    return this.$store.getters.findCategoryById;-->
<!--                },-->
<!--                findCategoryByName() {-->
<!--                    return this.$store.getters.findCategoryByName;-->
<!--                },-->
<!--                getSVGurl() {-->
<!--                    return "https://www.mallmaverick.com" + this.property.svg_map_url;-->
<!--                },-->
<!--                storeNames () {-->
<!--                    return _.map(this.filteredStores, 'name');-->
<!--                },-->
<!--                svgMapRef() {-->
<!--                    var reference = null; -->
<!--                    if(this.windowWidth <= 768) {-->
<!--                        reference = this.$refs.svgmaprefmobile;-->
<!--                    }-->
<!--                    else {-->
<!--                        reference = this.$refs.svgmap_ref;-->
<!--                    }-->
<!--                    return reference;-->
<!--                },-->
<!--                filterStores() {-->
<!--                    letter = this.selectedAlpha;-->
<!--                    if (letter == "All") {-->
<!--                        this.filteredStores = this.allStores;-->
<!--                    } else {-->
<!--                        var filtered = _.filter(this.allStores, function(o, i) {-->
<!--                            return _.lowerCase(o.name)[0] == _.lowerCase(letter);-->
<!--                        });-->
<!--                        this.filteredStores = filtered;-->
<!--                    }-->
<!--                },-->
<!--                floorList () {-->
<!--                    var floor_list = [];-->
                    
                    //get svg maps from repo
<!--                    var floor_maps_repo = this.findRepoByName('SVG Map');-->
                    
<!--                    if(floor_maps_repo !== null && floor_maps_repo !== undefined && floor_maps_repo.images.length > 0){-->
<!--                        floor_maps = floor_maps_repo.images;-->
<!--                        var floor_0 = {};-->
<!--                        floor_0.id = "basement-floor";-->
<!--                        floor_0.title = "Level 0";-->
<!--                        floor_0.map = _.find(floor_maps, function(o){ return _.toNumber(o.id) == _.toNumber(40469);}).image_url;-->
<!--                        floor_0.z_index = 0;-->
<!--                        floor_0.show = true;-->
                        
<!--                        floor_list.push(floor_0);-->
                        
<!--                        var floor_1 = {};-->
<!--                        floor_1.id = "first-floor";-->
<!--                        floor_1.title = "Level 1";-->
<!--                        floor_1.map = _.find(floor_maps, function(o){ return _.toNumber(o.id) == _.toNumber(40470);}).image_url;-->
<!--                        floor_1.z_index = 1;-->
<!--                        floor_1.show = false;-->
                        
<!--                        floor_list.push(floor_1);-->
                        
<!--                        var floor_2 = {};-->
<!--                        floor_2.id = "second-floor";-->
<!--                        floor_2.title = "Level 2";-->
<!--                        floor_2.map = _.find(floor_maps, function(o){ return _.toNumber(o.id) == _.toNumber(40471);}).image_url;-->
<!--                        floor_2.z_index = 2;-->
<!--                        floor_2.show = false;-->
                        
<!--                        floor_list.push(floor_2);-->
<!--                    }-->
                    
<!--                    return floor_list;-->
<!--                }-->
                
<!--            },-->
<!--            beforeDestroy: function() {-->
<!--                window.removeEventListener('resize', this.getWindowWidth);-->
<!--            }-->
<!--        });-->
<!--    });-->
<!--</script>-->