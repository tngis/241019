<script setup lang="ts">
import { computed } from 'vue'

const props = withDefaults(defineProps<{ size?: string; color?: string; secondaryColor?: string; loading?: boolean }>(), {
    size: '20px',
    color: '#c3dff4',
    secondaryColor: '#112940',
    loading: false
})

const innerStyles = computed(() => {
    const size = parseInt(props.size || props.size)
    return {
        transform: `scale(${size / 60})`,
        '--bg-color': props.color || props.color,
        '--secondary-color': props.secondaryColor || props.secondaryColor
    }
})

const styles = computed(() => ({
    width: props.size || props.size,
    height: props.size || props.size
}))

</script>

<template>
    <div :style="styles" class="spinner spinner-origami">
        <div :style="innerStyles" class="spinner-inner loading dark:#c3dff4">
            <span class="slice"></span>
            <span class="slice"></span>
            <span class="slice"></span>
            <span class="slice"></span>
            <span class="slice"></span>
            <span class="slice"></span>
        </div>
    </div>
</template>

<style lang="scss">
@for $i from 1 through 6 {
    @keyframes origami-show-#{$i} {
        from {
            transform: rotateZ(60 * $i + deg) rotateY(-90deg) rotateX(0deg);
            border-left-color: var(--secondary-color);
        }
    }

    @keyframes origami-hide-#{$i} {
        to {
            transform: rotateZ(60 * $i + deg) rotateY(-90deg) rotateX(0deg);
            border-left-color: var(--secondary-color);
        }
    }

    @keyframes origami-cycle-#{$i} {
        $startIndex: $i * 5;
        $reverseIndex: (
            80 - $i * 5
        );

    #{$startIndex * 1%} {
        transform: rotateZ(60 * $i + deg) rotateY(90deg) rotateX(0deg);
        border-left-color: var(--secondary-color);
    }

    #{$startIndex + 5%},
    #{$reverseIndex * 1%} {
        transform: rotateZ(60 * $i + deg) rotateY(0) rotateX(0deg);
        border-left-color: var(--bg-color);
    }

    #{$reverseIndex + 5%},
    100% {
        transform: rotateZ(60 * $i + deg) rotateY(90deg) rotateX(0deg);
        border-left-color: var(--secondary-color);
    }
}
}

.spinner {
    display: flex;
    justify-content: center;
    align-items: center;

    * {
        line-height: 0;
        box-sizing: border-box;
    }
}

.spinner-inner {
    display: block;
    width: 60px;
    height: 68px;

    .slice {
        border-top: 18px solid transparent;
        border-right: none;
        border-bottom: 16px solid transparent;
        border-left: 30px solid #f7484e;
        position: absolute;
        top: 0px;
        left: 50%;
        transform-origin: left bottom;
        border-radius: 3px 3px 0 0;
    }

    @for $i from 1 through 6 {
        .slice:nth-child(#{$i}) {
            transform: rotateZ(60 * $i + deg) rotateY(0deg) rotateX(0);
            animation: 0.15s linear 0.9 - $i * 0.08s origami-hide-#{$i} both 1;
        }
    }

    &.loading {
        @for $i from 1 through 6 {
            .slice:nth-child(#{$i}) {
                transform: rotateZ(60 * $i + deg) rotateY(90deg) rotateX(0);
                animation: 2s origami-cycle-#{$i} linear infinite both;
            }
        }
    }
}
</style>
