<template>
	<select :id="selectID" :class="selectClass" :multiple="multiple">
		<slot></slot>
	</select>
</template>

<script>
	export default {
		props: {
			options: [Array, String], 
			value: {
				type: [String, Number, Array],
				default: ''
			},
			placeholder: String, 
			allowClear: {
				type: Boolean,
				default: false
			}, 
			selectID: [String, Number],
			selectClass: [String, Number],
			selectOnClose: {
				type: Boolean,
				default: false
			},
			multiple: {
				type: Boolean,
				default: false
			},
			imgPath: [String],
			width: {
				type: String,
				default: '100%'
			},
			resize: {
				type: String,
				default: 'auto'
			}
		},

		mounted() {
			var vm = this;		

			this.initSelect2();
		},

		data() {
			return {
				select2: null
			}
		},

		computed: {
			optionsData() {
				var vm = this;

				// update options
	            let data = $.map(this.options, function (obj) {
	              obj.text = obj.text || obj.name; // replace name with the property used for the text

	              return obj;
	            });

	            return data;
			}
		},

		methods: {
			initSelect2() {
				// console.log("Init");
				
				let vm = this;

				this.select2 = $(this.$el).select2({ 
					data: this.optionsData,
					dropdownAutoWidth : true,
					width: this.width,
					placeholder: this.placeholder,
					allowClear: this.allowClear,
					selectOnClose: this.selectOnClose,
					templateResult: this.formatState
				})
				.val(this.value)
				.trigger('change.select2')
				.on('change', function () {
					if (this.multiple) {
						let dataArray = [];

						$(this).select2('data').forEach( (element) => {
							dataArray.push(element.id);
						})

						vm.$emit('input', dataArray);
					}
					else {
						vm.$emit('input', this.value);
					}
				});
			},

			formatState (state) {
				// console.log(state);
					if (!state.id || !state.image) {
						return state.text;
					}
					
					var baseUrl = "/" + this.imgPath + state.image;
					var $state = $(
						`<div class="select2flex">
							<img width="50px" src="` + baseUrl + `" />
							<div>` + state.text + `</div>
						</div>`
					);

					return $state;
			}
		},

		watch: {
			options(newVal, oldVal) {
				$(this.$el).empty();

				$(this.$el).select2({
					data: this.optionsData,
					dropdownAutoWidth : true,
					width: this.width,
					placeholder: this.placeholder,
					allowClear: this.allowClear,
					selectOnClose: this.selectOnClose,
					templateResult: this.formatState
				})
				.val(this.value)
				.trigger('change.select2');
			},

			value: function (value) {
				// console.log(value);
				let vm = this;

				vm.select2
				.val(this.value)
				.trigger('change.select2');
			}
		},

		destroyed() {
			$(this.$el).off().select2('destroy');
		}
	}
</script>

<style>
    .select2flex {
      display: flex;
      align-items: flex-start;
    }

    .select2flex > div {
      margin: 10px;
      text-align: center;
    }

	.select2-container .select2-selection--single,
	.select2-container .select2-selection--multiple {
		min-height: 28px;
		font-size: 13px;
	}

	.select2-results__options {
		font-size: 13px;
	}
</style>