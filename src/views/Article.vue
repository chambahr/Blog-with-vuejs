<template>
	<div class="container my-5">
		<div class="row">
			<div class="col-md-8 offset-md-2">
				<div class="card" v-if="!loading">
					<img height="420" :src="article.imageUrl" class="card-img-top">
					<div class="card-body">
						<h1 class="card-title text-center my-3">{{ article.title }}</h1>
						<div class="article-content" v-html="article.content"></div>
						<div class="comments my-4">
							<vue-disqus shortname="test-blog" :identifier="article.slug" :url="url"></vue-disqus>
						</div>
					</div>
				</div>
				<div class="loader text-center" v-else>
					<i class="fa fa-5x fa-spin fa-spinner"></i>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import Axios from 'axios';
import config from '@/config'
	export default {
		mounted() {
			this.getArticle()
		},
		data() {
			return {
				article: {},
				loading: true,
				url: window.location.href
			}
		},
		methods: {
			getArticle() {
				Axios.get(`${ config.apiUrl }/article/${this.$route.params.slug}`).then((response) => {
					console.log(response);
					this.loading = false;
					this.article = response.data.data;
				}).catch(({response}) => {
					console.log(response);
				});
			}
		}
	}
</script>