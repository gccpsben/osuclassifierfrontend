<template>
    <div style="width:500px; height:40px; position:relative;" class="center">
        <input :step="step" @input="$emit('update:min', Number.parseFloat($event.target.value))" :value="min" type="range" :min="rangeMin" :max="rangeMax" id="slider-1" style="position:absolute; z-index: 2;"/>
        <div :style="{'left':sliderTrackLeft, 'width':sliderTrackWidth}" class="slider-track"></div>
        <div class="slider-track-bg"></div>
        <input :step="step" @input="$emit('update:max', Number.parseFloat($event.target.value))" :value="max" type="range" :min="rangeMin" :max="rangeMax" ref="slider2" id="slider-2" style="position:absolute; z-index: 2;"/>
    </div>
</template>

<script>
export default 
{
    props: 
    {
        'min':Number,
        "max":Number, 
        "step":
        {
            type:Number,
            default: 1
        },
        "rangeMin":
        {
            type:Number,
            default: 0
        },
        "rangeMax":
        {
            type:Number,
            default: 100
        }
    },
    emits: ['update:min', "update:max"],
	computed:
	{
		sliderTrackWidth() { return (this.max - this.min) / (this.rangeMax - this.rangeMin) * 100 + "%"; },
		sliderTrackLeft() { return (this.min) / (this.rangeMax - this.rangeMin) * 100 + "%"; }
	},
	watch:
	{
		min() 
        { 
            if (this.min > this.max)
            {
                this.$emit('update:min', Math.min(this.min, this.max));
            }
        },
		max() 
        { 
            if (this.min > this.max)
            {
                this.$emit('update:max', Math.max(this.min, this.max));
            }
        }
	}
}
</script>

<style lang="less" scoped>
.slider-track
{
    width: 100%;
    height: 5px;
    position: absolute;
    margin: auto;
	background-color: #FF66AA;
    top: 0;
    bottom: 0;
    z-index: 1;
    border-radius: 5px;
}

.slider-track-bg
{
    width: 100%;
    height: 5px;
    position: absolute;
    margin: auto;
	background:#FF66AA12;
    top: 0;
    bottom: 0;
    border-radius: 5px;
}

input[type="range"]
{
    -webkit-appearance: none; -moz-appearance: none; appearance: none;
    width: 100%;
    outline: none;
    position: absolute;
    margin: auto;
    top: 0;
    bottom: 0;
    background-color: transparent;
    pointer-events: none;
}

.rangeThumbStyle 
{
    appearance: none;
    -ms-appearance: none;
    -webkit-appearance: none;
    height: 0.9em;
    width: 0.9em;
    cursor: pointer;
    border-radius: 50%;
    background-color: #FF66AA;
    border:0px;
    pointer-events: auto;
}

input[type="range"]::-webkit-slider-thumb { .rangeThumbStyle; }
input[type="range"]::-moz-range-thumb { .rangeThumbStyle; }
input[type="range"]::-ms-range-thumb { .rangeThumbStyle; }
</style>