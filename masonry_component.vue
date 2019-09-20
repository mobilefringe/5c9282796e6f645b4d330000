<template>
    <div v-masonry transition-duration="0.3s" item-selector=".grid-item">
        <div v-masonry-tile class="item" >
            <div v-for="feature in featurelist" :class="'grid-item ' + feature.masonry_class ">
            	<div  :class="{ 'ih-item circle effect19' : feature.no_hover_class}">
            	     <a v-if="feature.do_anchor_tag" :href="feature.url" target="_blank">
            			<img v-lazy="feature.image_url" :alt="'Click to read about' + feature.name">
            			<div class="info">
            				<div class="content">
            					<h3 v-if="locale=='en-ca' && feature.name.length > 0">{{feature.name}}</h3>
            					<h3 v-else-if="locale !=='en-ca' && feature.name_2.length > 0">{{feature.name_2}}</h3>
            				</div>
            			</div>
            		</a>
            		<router-link :to="feature.url" v-else>
            			<img v-lazy="feature.image_url" :alt="'Click to read about' + feature.name">
            			<div class="info">
            				<div class="content">
            					<h3 v-if="locale=='en-ca' && feature.name.length > 0"><span v-if="">{{feature.name}} </span></h3>
            					<h3 v-else-if="locale !=='en-ca' && feature.name_2.length > 0">{{feature.name_2}}</h3>
            				</div>
            			</div>
            		</router-link>
            	</div>
            </div>
        </div>
    </div>
</template>

<script>
    define(["Vue", "vuex", 'masonry' , 'vue-masonry-plugin'], function(Vue, Vuex, masonry, VueMasonryPlugin) {
        Vue.use(VueMasonryPlugin.default);
        return Vue.component("masonry-component", {
            template: template, // the variable template will be injected,
            props: {
                featurelist : {
                    type: Array,
                    required: true
                },
                locale: {
                    type: String
                },
            }
        });
    });
</script>