<template lang="pug">
	v-container(fluid, align-content-start, justify-start)
		v-layout
			v-flex(xs12)
				v-card
					v-card-title.title.pa-3 Редактирование сертификата {{ title }} лица
					legal-entities-form(v-if="itemCertificate.enityType === 'legal'",
															:parameters='itemCertificate',
															:files='files',
															@setFiles='setFiles',
															@deleteFile='deleteFile',
															@submit='updateCertificate')
					individuals-form(v-if="itemCertificate.enityType === 'individual'",
													:parameters='itemCertificate',
													:files='addedFiles',
													@setFiles='setFiles',
													@deleteFile='deleteFile',
													@submit='updateCertificate')
</template>

<script>
import axios from 'axios'
import { mapGetters } from 'vuex'

export default {
	name: 'EditCertificate',
	data () {
		return {
			addedFiles: []
		}
	},
	components: {
		LegalEntitiesForm: () => import('@/assets/components/forms/LegalEntitiesForm'),
		IndividualsForm: () => import('@/assets/components/forms/IndividualsForm')
	},
	computed: {
		...mapGetters(['files', 'fileCertificate']),
		itemCertificate () {
			return this.$store.state.itemSelectedCertificate
		},
		title () {
			return this.itemCertificate.enityType === 'legal' ? 'юридического' : 'физического'
		}
	},
	methods: {
		async updateCertificate (parameters) {
			try {
				await axios.post(`zxc/certificates.update`, this.filePreparation(parameters), { headers: { 'Content-Type': 'multipart/form-data' } })
				this.$router.go(-1)
				this.$store.commit('showSuccessMessage', 'Сертификат успешно отредактирован')
			} catch (err) {
				this.$store.commit('setError', err)
				this.$store.commit('showErorrMessage', 'Произошла ошибка при редактировании сертификата')
			}
		},
		filePreparation (parameters) {
			const formData = new FormData()
			let attachedFiles = []
			for (let i = 0; i < this.addedFiles.length; i++) {
				if ('lastModified' in this.addedFiles[i]) {
					continue
				} else {
					attachedFiles.push(this.addedFiles[i])
				}
			}
			parameters.attachedFiles = attachedFiles

			formData.append('parameters', JSON.stringify(parameters))
			for (let i = 0; i < this.addedFiles.length; i++) {
				if ('lastModified' in this.addedFiles[i]) {
					formData.append('files', this.addedFiles[i])
				} else {
					continue
				}
			}
			if ('lastModified' in this.fileCertificate) {
				formData.append('cert', this.fileCertificate)
			}
			return formData
		},
		setFiles (files) {
			for (let i = 0; i < files.length; i++) {
				this.addedFiles.push(files[i])
			}
		},
		deleteFile (fileIndex) {
			this.addedFiles.splice(fileIndex, 1)
		},
		setAddedFiles () {
			this.addedFiles = this.files
		}
	},
	mounted () {
		this.setAddedFiles()
	}
}
</script>
