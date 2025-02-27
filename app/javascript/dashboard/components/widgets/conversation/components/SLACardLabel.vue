<template>
  <div
    v-if="hasSlaThreshold"
    class="relative flex items-center border cursor-pointer min-w-fit border-slate-100 dark:border-slate-700"
    :class="showExtendedInfo ? 'h-[26px] rounded-lg' : 'rounded h-5'"
  >
    <div
      class="flex items-center w-full truncate"
      :class="showExtendedInfo ? 'px-1.5' : 'px-2 gap-1'"
      @mouseover="openSlaPopover()"
      @mouseleave="closeSlaPopover()"
    >
      <div
        class="flex items-center gap-1"
        :class="
          showExtendedInfo &&
          'ltr:pr-1.5 rtl:pl-1.5 ltr:border-r rtl:border-l border-solid border-slate-100 dark:border-slate-700'
        "
      >
        <fluent-icon
          size="14"
          :icon="slaStatus.icon"
          type="outline"
          :icon-lib="isSlaMissed ? 'lucide' : 'fluent'"
          class="flex-shrink-0"
          :class="slaTextStyles"
        />
        <span
          v-if="showExtendedInfo"
          class="text-xs font-medium"
          :class="slaTextStyles"
        >
          {{ slaStatusText }}
        </span>
      </div>
      <span
        class="text-xs font-medium"
        :class="[slaTextStyles, showExtendedInfo && 'ltr:pl-1.5 rtl:pr-1.5']"
      >
        {{ slaStatus.threshold }}
      </span>
    </div>
    <SLA-popover-card
      v-if="showSlaPopoverCard"
      :all-missed-slas="slaEvents"
      class="right-0 top-7"
    />
  </div>
</template>

<script>
import { evaluateSLAStatus } from '../helpers/SLAHelper';
import SLAPopoverCard from './SLAPopoverCard.vue';

const REFRESH_INTERVAL = 60000;

export default {
  components: {
    SLAPopoverCard,
  },
  props: {
    chat: {
      type: Object,
      default: () => ({}),
    },
    showExtendedInfo: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      timer: null,
      showSlaPopover: false,
      slaStatus: {
        threshold: null,
        isSlaMissed: false,
        type: null,
        icon: null,
      },
    };
  },
  computed: {
    slaPolicyId() {
      return this.chat?.sla_policy_id;
    },
    appliedSLA() {
      return this.chat?.applied_sla;
    },
    slaEvents() {
      return this.chat?.sla_events;
    },
    hasSlaThreshold() {
      return this.slaStatus?.threshold;
    },
    isSlaMissed() {
      return this.slaStatus?.isSlaMissed;
    },
    slaTextStyles() {
      return this.isSlaMissed
        ? 'text-red-400 dark:text-red-300'
        : 'text-yellow-600 dark:text-yellow-500';
    },
    slaStatusText() {
      const upperCaseType = this.slaStatus?.type?.toUpperCase(); // FRT, NRT, or RT
      const statusKey = this.isSlaMissed ? 'MISSED' : 'DUE';

      return this.$t(`CONVERSATION.HEADER.SLA_STATUS.${upperCaseType}`, {
        status: this.$t(`CONVERSATION.HEADER.SLA_STATUS.${statusKey}`),
      });
    },
    showSlaPopoverCard() {
      return (
        this.showExtendedInfo && this.showSlaPopover && this.slaEvents.length
      );
    },
  },
  watch: {
    chat() {
      this.updateSlaStatus();
    },
  },
  mounted() {
    this.updateSlaStatus();
    this.createTimer();
  },
  beforeDestroy() {
    if (this.timer) {
      clearTimeout(this.timer);
    }
  },
  methods: {
    createTimer() {
      this.timer = setTimeout(() => {
        this.updateSlaStatus();
        this.createTimer();
      }, REFRESH_INTERVAL);
    },
    updateSlaStatus() {
      this.slaStatus = evaluateSLAStatus(this.appliedSLA, this.chat);
    },
    openSlaPopover() {
      if (!this.showExtendedInfo) return;
      this.showSlaPopover = true;
    },
    closeSlaPopover() {
      this.showSlaPopover = false;
    },
  },
};
</script>
