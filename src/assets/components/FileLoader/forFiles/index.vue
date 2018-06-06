<template lang="pug">
	v-card.pa-2(color="grey lighten-4")
		v-card-title.title Прикрепление файлов
			v-layout
				v-flex
					input(type='file', name='file', multiple, ref='fileLoader', style='display: none', @change="setFiles($event)")
					v-btn(color='teal darken-2', small, dark, @click.stop='addFile') Добавить файлы
						v-icon(dark) import_export
		v-layout(v-if='files.length')
			v-flex
				v-list(subheader)
					v-subheader Добавленные файлы
					v-list-tile(v-for='(file, index) in files',
											:key='file.name',
											avatar)
						v-list-tile-avatar(style='min-width: 30px')
							v-icon(color='grey darken-2') insert_drive_file
						v-list-tile-content
							v-list-tile-title {{ file.name }}
						v-list-tile-action
							v-btn(icon, ripple, target='_blank', :href='file.url', v-if='file.url')
								v-icon(color='green lighten-1') save
						v-list-tile-action(v-if='canDelete')
							v-btn(icon, ripple, @click='deleteFile(index)')
								v-icon(color='red lighten-1') delete
</template>

<script>
import { mapGetters } from 'vuex'

export default {
	name: 'FileLoaderForFiles',
	props: {
		files: {
			type: Array
		}
	},
	computed: {
		...mapGetters(['role']),
		canDelete () {
			if (this.role === 'admin' || this.role === 'manager') {
				return true
			} else {
				return false
			}
		}
	},
	methods: {
		addFile () {
			this.$refs.fileLoader.click()
		},
		setFiles (e) {
			this.$emit('setFiles', e.target.files)
		},
		deleteFile (index) {
			this.$emit('deleteFile', index)
		}
	}
}
</script>
