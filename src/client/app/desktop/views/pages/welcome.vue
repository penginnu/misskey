<template>
<div class="mk-welcome">
	<button @click="dark">
		<template v-if="$store.state.device.darkmode">%fa:moon%</template>
		<template v-else>%fa:R moon%</template>
	</button>

	<mk-forkit class="forkit"/>

	<main>
		<div class="body">
			<div class="main block">
				<div>
					<h1 v-if="name != 'Misskey'">{{ name }}</h1>
					<h1 v-else><img :src="$store.state.device.darkmode ? 'assets/title.dark.svg' : 'assets/title.light.svg'" :alt="name"></h1>

					<div class="info">
						<span><b>{{ host }}</b> - <span v-html="'%i18n:@powered-by-misskey%'"></span></span>
						<span class="stats" v-if="stats">
							<span>%fa:user% {{ stats.originalUsersCount | number }}</span>
							<span>%fa:pencil-alt% {{ stats.originalNotesCount | number }}</span>
						</span>
					</div>

					<p class="desc" v-html="description || '%i18n:common.about%'"></p>
					<a class="about" @click="about">%i18n:@about%</a>

					<p class="sign">
						<span class="signup" @click="signup">%i18n:@signup%</span>
						<span class="divider">|</span>
						<span class="signin" @click="signin">%i18n:@signin%</span>
					</p>

					<img src="/assets/pointer.png" alt="" class="char">
				</div>
			</div>

			<div class="announcements block">
				<header>%fa:broadcast-tower% %i18n:@announcements%</header>
				<div v-if="announcements && announcements.length > 0">
					<div v-for="announcement in announcements">
						<h1 v-html="announcement.title"></h1>
						<div v-html="announcement.text"></div>
					</div>
				</div>
			</div>

			<div class="photos block">
				<header>%fa:images% %i18n:@photos%</header>
				<div>
					<div v-for="photo in photos" :style="`background-image: url(${photo.thumbnailUrl})`"></div>
				</div>
			</div>

			<div class="tag-cloud block">
				<div>
					<mk-tag-cloud/>
				</div>
			</div>

			<div class="nav block">
				<div>
					<mk-nav class="nav"/>
				</div>
			</div>

			<div class="side">
				<div class="trends block">
					<div>
						<mk-trends/>
					</div>
				</div>

				<div class="tl block">
					<header>%fa:comment-alt R% %i18n:@timeline%</header>
					<div>
						<mk-welcome-timeline class="tl" :max="20"/>
					</div>
				</div>

				<div class="info block">
					<header>%fa:info-circle% %i18n:@info%</header>
					<div>
						<div v-if="meta" class="body">
							<p>Version: <b>{{ meta.version }}</b></p>
							<p>Maintainer: <b><a :href="meta.maintainer.url" target="_blank">{{ meta.maintainer.name }}</a></b></p>
						</div>
					</div>
				</div>
			</div>
		</div>
	</main>

	<modal name="about" :class="$store.state.device.darkmode ? ['about', 'modal-dark'] : ['about', 'modal-light']" width="800px" height="auto" scrollable>
		<article class="fpdezooorhntlzyeszemrsqdlgbysvxq">
			<h1>%i18n:common.intro.title%</h1>
			<p v-html="'%i18n:common.intro.about%'"></p>
			<section>
				<h2>%i18n:common.intro.features%</h2>
				<section>
					<div class="body">
						<h3>%i18n:common.intro.rich-contents%</h3>
						<p v-html="'%i18n:common.intro.rich-contents-desc%'"></p>
					</div>
					<div class="image"><img src="/assets/about/post.png" alt=""></div>
				</section>
				<section>
					<div class="body">
						<h3>%i18n:common.intro.reaction%</h3>
						<p v-html="'%i18n:common.intro.reaction-desc%'"></p>
					</div>
					<div class="image"><img src="/assets/about/reaction.png" alt=""></div>
				</section>
				<section>
					<div class="body">
						<h3>%i18n:common.intro.ui%</h3>
						<p v-html="'%i18n:common.intro.ui-desc%'"></p>
					</div>
					<div class="image"><img src="/assets/about/ui.png" alt=""></div>
				</section>
				<section>
					<div class="body">
						<h3>%i18n:common.intro.drive%</h3>
						<p v-html="'%i18n:common.intro.drive-desc%'"></p>
					</div>
					<div class="image"><img src="/assets/about/drive.png" alt=""></div>
				</section>
			</section>
			<p v-html="'%i18n:common.intro.outro%'"></p>
		</article>
	</modal>

	<modal name="signup" :class="$store.state.device.darkmode ? 'modal-dark' : 'modal-light'" width="450px" height="auto" scrollable>
		<header class="formHeader">%i18n:@signup%</header>
		<mk-signup class="form"/>
	</modal>

	<modal name="signin" :class="$store.state.device.darkmode ? 'modal-dark' : 'modal-light'" width="450px" height="auto" scrollable>
		<header class="formHeader">%i18n:@signin%</header>
		<mk-signin class="form"/>
	</modal>
</div>
</template>

<script lang="ts">
import Vue from 'vue';
import { host, copyright } from '../../../config';
import { concat } from '../../../../../prelude/array';

export default Vue.extend({
	data() {
		return {
			meta: null,
			stats: null,
			copyright,
			host,
			name: 'Misskey',
			description: '',
			announcements: [],
			photos: []
		};
	},

	created() {
		(this as any).os.getMeta().then(meta => {
			this.meta = meta;
			this.name = meta.name;
			this.description = meta.description;
			this.announcements = meta.broadcasts;
		});

		(this as any).api('stats').then(stats => {
			this.stats = stats;
		});

		const image = [
			'image/jpeg',
			'image/png',
			'image/gif'
		];

		(this as any).api('notes/local-timeline', {
			fileType: image,
			limit: 6
		}).then((notes: any[]) => {
			const files = concat(notes.map((n: any): any[] => n.files));
			this.photos = files.filter(f => image.includes(f.type)).slice(0, 6);
		});
	},

	methods: {
		about() {
			this.$modal.show('about');
		},

		signup() {
			this.$modal.show('signup');
		},

		signin() {
			this.$modal.show('signin');
		},

		dark() {
			this.$store.commit('device/set', {
				key: 'darkmode',
				value: !this.$store.state.device.darkmode
			});
		}
	}
});
</script>

<style lang="stylus">
#wait
	right auto
	left 15px

.v--modal-overlay
	background rgba(0, 0, 0, 0.6)

.modal-light
	.v--modal-box
		color #777

		.formHeader
			border-bottom solid 1px #eee

.modal-dark
	.v--modal-box
		background #313543
		color #fff

		.formHeader
			border-bottom solid 1px rgba(#000, 0.2)

.modal-light
.modal-dark
	.form
		padding 24px 48px 48px 48px

	.formHeader
		text-align center
		padding 48px 0 12px 0
		margin 0 48px
		font-size 1.5em

.v--modal-overlay.about
	.v--modal-box.v--modal
		margin 32px 0

.fpdezooorhntlzyeszemrsqdlgbysvxq
	padding 64px

	> p:last-child
		margin-bottom 0

	> h1
		margin-top 0

	> section
		> h2
			border-bottom 1px solid isDark ? rgba(#000, 0.2) : rgba(#000, 0.05)

		> section
			display grid
			grid-template-rows 1fr
			grid-template-columns 180px 1fr
			gap 32px
			margin-bottom 32px
			padding-bottom 32px
			border-bottom 1px solid isDark ? rgba(#000, 0.2) : rgba(#000, 0.05)

			&:nth-child(odd)
				grid-template-columns 1fr 180px

				> .body
					grid-column 1

				> .image
					grid-column 2

			> .body
				grid-row 1
				grid-column 2

			> .image
				grid-row 1
				grid-column 1

				> img
					display block
					width 100%
					height 100%
					object-fit cover
</style>

<style lang="stylus" scoped>
@import '~const.styl'

root(isDark)
	display flex
	min-height 100vh
	//background-color #00070F
	//background-image url('/assets/bg.jpg')
	//background-position center
	//background-size cover

	> .forkit
		position absolute
		top 0
		right 0

	> button
		position fixed
		z-index 1
		bottom 16px
		left 16px
		padding 16px
		font-size 18px
		color isDark ? #fff : #444

	> main
		margin 0 auto
		padding 64px
		width 100%
		max-width 1200px

		.block
			color isDark ? #fff : #444
			background isDark ? #282C37 : #fff
			box-shadow 0 3px 8px rgba(0, 0, 0, 0.2)
			//border-radius 8px
			overflow auto

			> header
				z-index 1
				padding 0 16px
				line-height 48px
				background isDark ? #313543 : #fff

				if !isDark
					box-shadow 0 1px 0px rgba(0, 0, 0, 0.1)

				& + div
					max-height calc(100% - 48px)

			> div
				overflow auto

		> .body
			display grid
			grid-template-rows 1fr 1fr 256px 64px
			grid-template-columns 1fr 1fr 350px
			gap 16px
			height 1200px

			> .main
				grid-row 1
				grid-column 1 / 3
				border-top solid 5px $theme-color

				> div
					padding 32px
					min-height 100%

					> h1
						margin 0

						> img
							margin -8px 0 0 -16px
							max-width 280px

					> .info
						margin 0 auto 16px auto
						width $width
						font-size 14px

						> .stats
							margin-left 16px
							padding-left 16px
							border-left solid 1px isDark ? #fff : #444

							> *
								margin-right 16px

					> .sign
						font-size 120%

						> .divider
							margin 0 16px

						> .signin
						> .signup
							cursor pointer

							&:hover
								color $theme-color

					> .char
						display block
						position absolute
						right 16px
						bottom 16px
						width 180px
						opacity 0.3

					> *:not(.char)
						z-index 1

			> .announcements
				grid-row 2
				grid-column 1

				> div
					padding 32px

					> div
						padding 0 0 16px 0
						margin 0 0 16px 0
						border-bottom 1px solid isDark ? rgba(#000, 0.2) : rgba(#000, 0.05)

						> h1
							margin 0
							font-size 1.25em

			> .photos
				grid-row 2
				grid-column 2

				> div
					display grid
					grid-template-rows 1fr 1fr 1fr
					grid-template-columns 1fr 1fr
					gap 8px
					height 100%
					padding 16px

					> div
						//border-radius 4px
						background-position center center
						background-size cover

			> .tag-cloud
				grid-row 3
				grid-column 1 / 3

				> div
					height 256px
					padding 32px

			> .nav
				display flex
				justify-content center
				align-items center
				grid-row 4
				grid-column 1 / 3
				font-size 14px

			> .side
				display grid
				grid-row 1 / 5
				grid-column 3
				grid-template-rows 1fr 350px
				grid-template-columns 1fr
				gap 16px

				> .tl
					grid-row 1
					grid-column 1
					overflow auto

				> .trends
					grid-row 2
					grid-column 1
					padding 8px

				> .info
					grid-row 3
					grid-column 1

					> div
						padding 16px

						> .body
							> p
								display block
								margin 0

.mk-welcome[data-darkmode]
	root(true)

.mk-welcome:not([data-darkmode])
	root(false)

</style>
