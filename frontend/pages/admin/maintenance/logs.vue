<template>
  <v-container fluid>
    <BaseCardSectionTitle class="pb-0" :icon="$globals.icons.cog" title="$t('admin.maintenance.summary-title')">
    </BaseCardSectionTitle>
    <div class="mb-6 ml-2 d-flex" style="gap: 0.8rem">
      <BaseButton color="info" :loading="state.loading" @click="refreshLogs">
        <template #icon> {{ $globals.icons.refreshCircle }} </template>
        {{ $t("admin.maintenance.logs-action-refresh") }}
      </BaseButton>
      <AppButtonCopy :copy-text="copyText" />
      <div class="ml-auto" style="max-width: 150px">
        <v-text-field
          v-model="state.lines"
          type="number"
          label="$t('admin.maintenance.logs-tail-lines-label')"
          hide-details
          dense
          outlined
        >
        </v-text-field>
      </div>
    </div>
    <v-card outlined>
      <v-virtual-scroll
        v-scroll="scrollOptions"
        :bench="20"
        :items="logs.logs"
        height="800"
        item-height="20"
        class="keep-whitespace log-container"
      >
        <template #default="{ item }">
          <p class="log-text">
            {{ item }}
          </p>
        </template>
      </v-virtual-scroll>
    </v-card>
  </v-container>
</template>

<script lang="ts">
import { defineComponent, ref, computed, onMounted, reactive } from "@nuxtjs/composition-api";
import { useAdminApi } from "~/composables/api";

export default defineComponent({
  layout: "admin",
  setup() {
    const adminApi = useAdminApi();

    const state = reactive({
      loading: false,
      lines: 500,
      autoRefresh: true,
    });

    const scrollOptions = reactive({
      enable: true,
      always: false,
      smooth: false,
      notSmoothOnInit: true,
    });

    const logs = ref({
      logs: [] as string[],
    });

    async function refreshLogs() {
      state.loading = true;
      const { data } = await adminApi.maintenance.logs(state.lines);
      if (data) {
        logs.value = data;
      }
      state.loading = false;
    }
    onMounted(() => {
      refreshLogs();
    });

    const copyText = computed(() => {
      return logs.value.logs.join("") || "";
    });
    return {
      copyText,
      scrollOptions,
      state,
      refreshLogs,
      logs,
    };
  },
  head() {
    return {
      title: this.$t("admin.maintenance.logs-page-title") as string,
    };
  },
});
</script>

<style>
.log-text {
  font: 0.8rem Inconsolata, monospace;
}
.log-container {
  background-color: var(--v-background-base) !important;
}
.keep-whitespace {
  white-space: pre;
}
</style>
