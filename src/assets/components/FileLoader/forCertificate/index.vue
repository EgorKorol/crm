<template lang="pug">
	v-card.pa-2(color="grey lighten-4")
		v-card-title.title Прикрепление сертификата
			v-layout
				v-flex
					input(type='file', name='file', ref='fileLoader', style='display: none', @change="setFiles($event)")
					v-btn(v-if='!fileCertificate', color='teal darken-2', small, dark, @click.stop='addFile') Добавить файл
						v-icon(dark) import_export
		v-layout(v-if='fileCertificate')
			v-flex
				v-list(subheader)
					v-subheader Добавленный сертификат
					v-list-tile(avatar)
						v-list-tile-avatar(style='min-width: 30px')
							v-icon(color='grey darken-2') insert_drive_file
						v-list-tile-content
							v-list-tile-title {{ fileCertificate.name }}
						v-list-tile-action
							v-btn(icon, ripple, target='_blank', :href='fileCertificate.url', v-if='fileCertificate.url')
								v-icon(color='green lighten-1') save
						//v-list-tile-action(v-if='canDelete')
							v-btn(icon, ripple, @click='deleteFile')
								v-icon(color='red lighten-1') delete
</template>

<script>
import { mapGetters } from 'vuex'

export default {
	name: 'FileLoaderForCertificate',
	computed: {
		...mapGetters(['role', 'fileCertificate'])
	},
	methods: {
		addFile () {
			this.$refs.fileLoader.click()
		},
		setFiles (e) {
			this.$store.commit('setFileCertificate', e.target.files[0])
		}
	}
}
</script>
