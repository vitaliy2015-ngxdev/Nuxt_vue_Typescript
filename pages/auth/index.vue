<script lang="ts">
import helpers from '@includes/helpers'
import { Context } from '@nuxt/types'
import { Vue, Component } from 'nuxt-property-decorator'

@Component({
	layout: 'no-layout',
})
export default class PageAuth extends Vue {
	user: any = false

	authenticate() {
		const {
			$api,
			$route: { query },
		} = this

		const auth_params: any = helpers.extract(query, [
			'time',
			'vendor',
			'user_id',
			'target_page',
			'value',
		])

		return $api
			.$post('/members/auth/', auth_params)
			.then((user) => {
				localStorage.setItem('access', user.token.access)
				localStorage.setItem('refresh', user.token.refresh)

				this.user = user
			})
			.catch((err) => console.error(err))
	}

	async mounted() {
		await this.authenticate()

		if (this.user && this.user.id) {
			window.location.href = '/'
		} else {
			alert('Failed to authenticate!')
		}
	}
}
</script>

<template>
	<loading-screen start-it>
		<h2>Authenticating</h2>
	</loading-screen>
</template>

<style lang="scss" scoped>
@import '@sassy';

.loading-screen::v-deep {
	z-index: 1000000000000000000000000000000000;

	.loading-icon-wrap {
		// margin-right: 2em;
	}
}
</style>
