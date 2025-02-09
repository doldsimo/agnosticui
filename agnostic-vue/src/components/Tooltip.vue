<template>
  <div
    ref="tooltipTargetRef"
    :aria-describedby="id"
    @focus="showTooltip"
    @blur="hideTooltip"
    @mouseenter="showTooltip"
    @mouseleave="hideTooltip"
  >
    <slot></slot>
  </div>
  <div
    :id="id"
    ref="tooltipRef"
    role="tooltip"
    :class="tooltipClasses"
  >
    <slot name="content"></slot>
    <div
      id="arrow"
      ref="tooltipArrowRef"
      :class="tooltipArrowClasses"
    ></div>
  </div>
</template>
<script setup lang="ts">
// TODO -- we need to consider nuking the arrow id

import { computed, ref, useCssModule } from "vue";
import { computePosition, flip, offset, shift, arrow } from "@floating-ui/dom";

export interface TooltipProps {
  placement:
    | "top"
    | "top-start"
    | "top-end"
    | "right"
    | "right-start"
    | "right-end"
    | "bottom"
    | "bottom-start"
    | "bottom-end"
    | "left"
    | "left-start"
    | "left-end";
  offset: number;
}

const props = withDefaults(defineProps<TooltipProps>(), {
  placement: "top",
  offset: 6,
});

const uid = () =>
  `tip-${Date.now().toString(32)}-${Math.random()
    .toString(16)
    .replace(/\./g, "")
    .toString()}`;

const id = uid();

const tooltipTargetRef = ref<HTMLDivElement | null>(null);
const tooltipRef = ref<HTMLDivElement | null>(null);
const tooltipArrowRef = ref<HTMLDivElement | null>(null);

const update = () => {
  const tip = tooltipRef?.value;
  const target = tooltipTargetRef?.value;
  const arrowEl = tooltipArrowRef?.value;

  if (tip && target && arrowEl) {
    computePosition(target, tip, {
      placement: props.placement,
      middleware: [
        offset(props.offset),
        flip(),
        shift({ padding: 4 }),
        arrow({
          element: arrowEl,
          // Offset from edge; fixes issue on left-start, left-end, right-start,
          // etc., where arrow oddly overlaps the rounded corner of content box
          padding: 10,
        }),
      ],
    }).then(({ x, y, placement, middlewareData }) => {
      const arrowX = middlewareData.arrow!.x;
      const arrowY = middlewareData.arrow!.y;
      const staticSide = {
        top: "bottom",
        right: "left",
        bottom: "top",
        left: "right",
      }[placement.split("-")[0]]!;

      Object.assign(tip.style, {
        left: `${x}px`,
        top: `${y}px`,
      });

      Object.assign(arrowEl.style, {
        left: typeof arrowX === "number" ? `${arrowX}px` : "",
        top: typeof arrowY === "number" ? `${arrowY}px` : "",
        right: "",
        bottom: "",
        [staticSide]: "-4px",
      });
    });
  }
};

const showTooltip = () => {
  if (tooltipRef?.value) {
    tooltipRef.value.style.display = "block";
  }
  update();
};

const hideTooltip = () => {
  if (tooltipRef?.value) {
    tooltipRef.value.style.display = "";
  }
};

const styles = useCssModule();

const tooltipClasses = computed(() => {
  return {
    [styles["tooltip"]]: true,
  };
});

const tooltipArrowClasses = computed(() => {
  return {
    [styles["tooltip-arrow"]]: true,
  };
});
</script>
<style module>
.tooltip {
  display: none;
  position: absolute;
  background: var(--agnostic-dark);
  color: var(--agnostic-light);
  padding: var(--fluid-4) var(--fluid-8);
  border-radius: var(--fluid-4);
  pointer-events: none;
}

.tooltip-arrow {
  position: absolute;
  background: var(--agnostic-dark);
  width: var(--fluid-8);
  height: var(--fluid-8);
  transform: rotate(45deg);
}

</style>
