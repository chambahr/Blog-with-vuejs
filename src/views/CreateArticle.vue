<template>
	<div class="container">
		<div class="row">
			<div class="col-md-8 offset-md-2">
				<div class="card my-5">
					<div class="card-body">
						<picture-input
						  class="mb-3" 
						  width="600" 
					      height="600" 
					      margin="16" 
					      accept="image/jpeg,image/png" 
					      size="5" 
					      button-class="btn btn-danger"
					      @change="onChange" />
					    <select class="form-control my-3" v-model="category">
					    	<option selected>Select a Category</option>
					    	<option :value="category.id" :key="category.id" v-for="category in categories">
					    		{{ category.name }}
					    	</option>
					    </select>
						<input type="text" placeholder="title" v-model="title" class="my-3 form-control" />
						<wysiwyg v-model="content" />

						<div class="text-center">
							<button @click="createArticle()" :disabled="loading" class="btn btn-success btn-lg mt-3">
								<i class="fa fa-spin fa-spinner" v-if="loading"></i>
								{{ loading ? '' : 'Create article' }}
							</button>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import PictureInput from 'vue-picture-input'
import Axios from 'axios';
import config from '@/config';

	export default{
		beforeRouteEnter(to, from, next) {
	 		console.log(to, from, next);
	 		if (!localStorage.getItem('auth')) {
	 			return next({  path: '/login' });
	 		}

	 		next();
	 	},
		mounted() {
			// console.log(process.env);
			this.getCategories();
		},
		data() {
			return {
				content: '',
				image: null,
				categories: [],
				category: '',
				title: '',
				loading: false
			}
		},
		components: {
			PictureInput
		},
		methods: {
			onChange(image) {
				console.log("new image selected");
				if (image) {
					this.image = image;
				} else {
					console.log('FileReader API not supported: use the <form>, Russell!');
					this.$noty.error('FileReader API not supported: use the <form>, Russell!');

				}
			},
			createArticle() {
				// pass a check to make sure all fields are filled before the submit button is clicked
				if (!this.title || !this.content || !this.category || !this.image) {	
					this.$noty.error('Please fill all fields');				
					return;
					
				}

				this.loading = true
				const form = new FormData();
				form.append('file', this.image);
				form.append('upload_preset', process.env.VUE_APP_CLOUDINARY_PRESET);
				form.append('api_key', process.env.VUE_APP_CLOUDINARY_API_KEY);

				Axios.post(process.env.VUE_APP_CLOUDINARY_URL, form)
					.then(res => {
						console.log(res);
						Axios.post(`${config.apiUrl}/articles`, {
							title: this.title,
							content: this.content,
							category_id: this.category,
							imageUrl: res.data.secure_url
						}, {
							// Passing an authorization token from the auth user;
							headers: {
								Authorization: `Bearer ${this.$root.auth.token}`
							}
						}).then(() => {
							// Loading state
							this.loading = false;
							this.$noty.success('Article created successfully');
							this.$router.push('/');
						}).catch(() => {
							// loading state
							this.loading = false;
							this.$noty.error('Oops! Something went wrong');
						});
					}).catch(() => {
						this.loading = false;
						this.$noty.error('Oops! Something went wrong');
					});
			},
			getCategories() {
				// after the api request, get the categories from local storage
				const categories = localStorage.getItem('categories');
				// Now check if any exist before proceeding
				if (categories) {
					this.categories = JSON.parse(categories);
					// return the data obtained
					return;
				}
				Axios.get(`${config.apiUrl}/categories`).then(res => {
					this.categories = res.data.categories;
					// Save the categories on the local storage
					localStorage.setItem("categories", JSON.stringify(this.categories));
				});
			}
		}
	}
</script>