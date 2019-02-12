<template>
    <div class="hours_page_container"> <!-- template will not render without an outer container -->
        <div class="inside_page_banner" :style="{ backgroundImage: 'url(' + hoursImage.image_url + ')' }"></div>
        <div class="page_container">
            <div class="row">
                <div class="col-md-12">
                    <div v-html="currentPage.body"></div>
                </div>
            </div>
        </div>
        <div class="margin_30"></div>
        <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d3315.8160866223798!2d-118.33180808482304!3d33.79124718067779!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x80dd4a44b52715a9%3A0xd7414911d196d0b7!2sRolling+Hills+Plaza!5e0!3m2!1sen!2sca!4v1549987085186" width="100%" height="300" frameborder="0" style="border:0" allowfullscreen></iframe>
    </div>
</template>
<style>
    .footer-div {
        margin-top: 0;
    }
</style>
<script>
    define(["Vue", "vuex"], function(Vue, Vuex) {
        return Vue.component("directions-component", {
            template: template, // the variable template will be injected
            data: function() {
                return {
                    hoursImage: null,
                    currentPage : null,
                }
            },
            created() {
                this.loadData().then(response => {
                    this.dataloaded = true;
                    
                    var temp_repo = this.findRepoByName('Hours Image');
                    if(temp_repo) {
                        this.hoursImage = temp_repo.images[0];
                    } else {
                        this.hoursImage = {};
                        this.hoursImage.image_url = "";
                    }
                    
                    this.currentPage = response[1].data;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName'
                ])
            },
            methods: {
                loadData: async function() {
                    try {
                        var host_name = this.property.mm_host.replace("http:", "");
                        let results = await Promise.all([ 
                            this.$store.dispatch("getData", "repos"),
                            this.$store.dispatch('LOAD_PAGE_DATA', { url: host_name + "/pages/rollinghills-directions.json" }), 
                        ]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
            }
        });
    });
</script>