<template>
	<ul class="list-urls">
		<li v-for="url in urls">
			<label class="radio">
				<input v-model="innerValue" :value="url" class="radio__input" type="radio" name="url">

				<span class="radio__label">{{url}}</span>
			</label>
		</li>

		<li class="list-urls__custom">
			<label class="radio">
				<input class="radio__input" type="radio" name="url" >

				<span class="radio__label">Custom</span>

				<span class="list-urls__custom-hidden">
					<input v-model="innerValue" class="field" type="text" name="url">
				</span>
			</label>
		</li>
	</ul>
</template>

<script>
import ListUrls from 'components/ListUrls.vue';

export default {
	name: 'list-urls',

	props: ['value'],

	data() {
		return {
			innerValue: this.value
		};
	},

	computed: {
		/**
		 * Get url suggestions.
		 * @return {Array}
		 */
		urls() {
			const origin = window.location.origin;
			const paths = window.location.pathname.replace(/\/$/, '').split('/');
			const urls = [];

			paths.forEach((path, index, arr) => {
				const url = origin + paths.slice(0, arr.length - index).join('/');
				urls.push(url);
			});

			return urls;
		},
	},

	mounted() {
		if (!this.value && this.urls.length) {
			this.innerValue = this.urls[0];
		}
	},

	watch: {
		innerValue() {
			this.$emit('input', this.innerValue);
		}
	}
}
</script>
