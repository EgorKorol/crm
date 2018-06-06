<template lang="pug">
	v-container(fluid, align-content-start, justify-start)
		v-layout
			v-flex(xs12)
				v-card.elevation-2
					v-card-title.title Сертификат {{ title }} лица
						v-spacer
						input(ref='inputFile' type="file", style='display: none', @change="parseExcel($event)")
						v-btn(color='teal darken-2', dark, @click='addFile') Импорт из Exсel
							v-icon(dark) import_export
					v-container(fill-height, fluid)
						v-layout
							v-flex(xs12)
								legal-entities-form(v-if="type === 'legal'",
																		:parameters='legalParameters',
																		:files='files',
																		@setFiles='setFiles',
																		@deleteFile='deleteFile',
																		@submit='sendSertificate')
								individuals-form(v-if="type === 'individual'",
																:parameters='individualParameters',
																:files='files',
																@setFiles='setFiles',
																@deleteFile='deleteFile',
																@submit='sendSertificate')
</template>

<script>
import axios from 'axios'
import XLSX from 'xlsx'
import moment from 'moment'

export default {
	name: 'NewCertificate',
	components: {
		LegalEntitiesForm: () => import('@/assets/components/forms/LegalEntitiesForm'),
		IndividualsForm: () => import('@/assets/components/forms/IndividualsForm')
	},
	props: {
		type: {
			type: String
		}
	},
	data () {
		return {
			active: '',
			excelJson: [],
			files: [],
			legalParameters: {
				fullName: '',
				post: '',
				inn: '',
				kpp: '',
				ogrn: '',
				companyName: '',
				fss: '',
				pfr: '',
				kno: '',
				snils: '',
				passport: '',
				bDate: '',
				bAddress: '',
				registrationAddress: '',
				email: '',
				phone: '',
				certType: '',
				groupType: '',
				numberInGroup: '',
				cryptoProvider: '',
				reasonMaking: '',
				issuance: '',
				status: '',
				executorsId: '',
				innGroup: '',
				installersId: '',
				clientId: '',
				note: '',
				enityType: this.$route.params.type
			},
			individualParameters: {
				fullName: '',
				passport: '',
				bDate: '',
				bAddress: '',
				registrationAddress: '',
				snils: '',
				inn: '',
				email: '',
				phone: '',
				innGroup: '',
				certType: '',
				groupType: '',
				numberInGroup: '',
				cryptoProvider: '',
				reasonMaking: '',
				issuance: '',
				status: '',
				executorsId: '',
				installersId: '',
				clientId: '',
				note: '',
				enityType: this.$route.params.type
			}
		}
	},
	computed: {
		title () {
			if (this.type === 'legal') {
				return 'юридического'
			}
			if (this.type === 'individual') {
				return 'физического'
			}
		}
	},
	methods: {
		async sendSertificate (parameters) {
			try {
				await this.addCertificate(parameters)
				this.$router.go(-1)
				this.$store.commit('showSuccessMessage', 'Сертификат успешно добавлен')
			} catch (err) {
				this.$store.commit('setError', err)
				this.$store.commit('showErorrMessage', 'Произошла ошибка при добавлении сертификата')
			}
		},
		async addCertificate (parameters) {
			await axios.post(`${this.$root.SERVER_HOST}/certificates.add`, this.filePreparation(parameters), { headers: { 'Content-Type': 'multipart/form-data' } })
		},
		filePreparation (parameters) {
			const formData = new FormData()
			formData.append('parameters', JSON.stringify(parameters))
			for (var i = 0; i < this.files.length; i++) {
				let file = this.files[i]
				formData.append('files', file)
			}
			return formData
		},
		setFiles (files) {
			for (let i = 0; i < files.length; i++) {
				this.files.push(files[i])
			}
		},
		deleteFile (fileIndex) {
			this.files.splice(fileIndex, 1)
		},
		addFile () {
			this.$refs.inputFile.click()
		},
		fixdata (data) {
			let o = ''
			let l = 0
			let w = 10240
			for (; l < data.byteLength / w; ++l) o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w, l * w + w)))
			o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w)))
			return o
		},
		parseExcel (e) {
			e.stopPropagation()
			e.preventDefault()
			let file = e.target.files[0]
			let reader = new FileReader()
			reader.onload = e => {
				let result
				let data = e.target.result
				let fixedData = this.fixdata(data)
				let workbook = XLSX.read(btoa(fixedData), {type: 'base64'})
				let firstSheetName = workbook.SheetNames[0]
				let worksheet = workbook.Sheets[firstSheetName]
				result = XLSX.utils.sheet_to_json(worksheet)
				console.log(result)
				if (result.length <= 32) {
					this.setLegalForm(result)
				} else if (result.length >= 56) {
					this.setIndividualForm(result)
				}
			}
			reader.readAsArrayBuffer(file)
		},
		setLegalForm (array) {
			this.legalParameters.fullName = array[11].__EMPTY_1
			this.legalParameters.post = array[12].__EMPTY_1
			this.legalParameters.inn = this.deleteSymbols(array[13].__EMPTY_1)

			let arr = Object.values(array[13])
			this.legalParameters.kpp = this.deleteSymbols(arr[3])

			this.legalParameters.ogrn = this.deleteSymbols(array[14].__EMPTY_1)
			/**
			проверка на пустые (необязательные поля)
			*/
			if ('__EMPTY_1' in array[15]) {
				this.legalParameters.companyName = array[15].__EMPTY_1
			}
			if ('__EMPTY_1' in array[16]) {
				this.legalParameters.fss = this.deleteSymbols(array[16].__EMPTY_1)
			}
			if ('__EMPTY_1' in array[17]) {
				this.legalParameters.pfr = this.deleteSymbols(array[17].__EMPTY_1)
			}
			this.legalParameters.kno = array[18].__EMPTY_1
			this.legalParameters.snils = this.deleteSymbols(array[19].__EMPTY_1)
			this.legalParameters.passport = array[20].__EMPTY_1
			this.legalParameters.bDate = moment(array[21].__EMPTY_1, 'M-D-YY').format('DD.MM.YYYY')
			this.legalParameters.bAddress = array[22].__EMPTY_1
			this.legalParameters.registrationAddress = array[23].__EMPTY_1
			this.legalParameters.email = array[24].__EMPTY_1
			this.legalParameters.phone = this.deleteSymbols(array[25].__EMPTY_1)
		},
		setIndividualForm (array) {
			this.individualParameters.fullName = array[2].__EMPTY

			const arr = Object.values(array[5])
			const issuanceCenter = arr[0]
			const datePassport = moment(array[4].__EMPTY_7, 'M-D-YY').format('DD.MM.YYYY')
			const passportValue = `${array[4].__EMPTY} ${array[4].__EMPTY_2} ${array[4].__EMPTY_5} ${datePassport} ${issuanceCenter}`

			this.individualParameters.passport = passportValue
			this.individualParameters.bDate = moment(array[7].__EMPTY, 'M-D-YY').format('DD.MM.YYYY')
			this.individualParameters.bAddress = array[7].__EMPTY_4
			this.individualParameters.registrationAddress = array[8].__EMPTY_1
			this.individualParameters.snils = this.deleteSymbols(array[12].__EMPTY_2)
			this.individualParameters.inn = this.deleteSymbols(array[13].__EMPTY_2)
			this.individualParameters.email = array[18].__EMPTY_2
			this.individualParameters.phone = this.deleteSymbols(array[26].__EMPTY_2)
		},
		deleteSymbols (string) {
			return string.replace(/\D/g, '')
		}
	}
}
</script>
