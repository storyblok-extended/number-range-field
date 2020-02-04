<template>
	<div>
		<input
			v-bind:class="{
				'uk-width-1-1': true,
				'uk-range': is_range,
				'uk-input': !is_range
			}"
			v-bind:style="is_range ? { paddingRight: padding + 'px', paddingLeft: '5px' } : ''"
			v-bind:placeholder="options.placeholder || null"
			v-bind:type="is_range ? 'range' : 'number'"
			v-bind:min="model.min"
			v-bind:max="model.max"
			v-bind:step="model.step"
			v-model.number="model.value"
		/>

		<label class="p-sbe-number-range_value" v-if="is_range">{{ model.value }}</label>
	</div>
</template>

<script>
export default {
	data() {
		return {
			is_range: false,
			padding: null
		};
	},
	mixins: [window.Storyblok.plugin],
	methods: {
		getDefaults() {
			return {
				plugin: 'sbe-number-range',
				step: 1,
				min: null,
				max: null,
				value: null
			};
		},
		validateInput(n, f) {
			// convert input number to actual number
			const cn = parseFloat(n);
			// if valid, return it
			if (typeof cn === 'number' && !isNaN(cn)) return cn;

			// otherwise return the fallback
			return f;
		},
		getComputed() {
			// console.log('sbe-number-range', 'plugin:computed', this.options);
			// defaults
			const defaults = this.getDefaults();

			// constraints checks are left to the browser
			const computed = {
				...defaults,
				step: this.validateInput(this.options.step, defaults.step),
				min: this.validateInput(this.options.min, defaults.min),
				max: this.validateInput(this.options.max, defaults.max),
				value: this.validateInput(this.options.value, defaults.value)
			};

			// check if should render as HTML5 range type
			this.is_range =
				(this.options.max &&
					this.options.min &&
					this.options.slider === 'true') ||
				false;

			// compute the padding if it is a range (slider)
			if (this.is_range) {
				const digits = computed.max.toFixed(0).toString().length - 1 || 1;
				const decimals = computed.step.toString().length - 1 || 1;
				const base = 25;
				const space = 6.5;
				this.padding = base + space * decimals + space * digits;
				computed.value = this.validateInput(computed.value, computed.min);
			}

			// return the computed
			return computed;
		},
		initWith() {
			// console.log('sbe-number-range', 'plugin:init', this.options);
			// if options are initialized
			if (this.options) return this.getComputed();

			// otherwise the defaults
			return this.getDefaults();
		},
		pluginCreated() {
			// console.log('sbe-number-range', 'plugin:created', this.options);
		}
	},
	watch: {
		model: {
			handler(value) {
				this.$emit('changed-model', value);
			},
			deep: true
		}
	}
};
</script>

<style>
.p-sbe-number-range_value {
	position: absolute;
	right: 5px;
}
</style>
