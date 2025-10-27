<template>
  <div>
    <v-card density="compact" variant="flat">
      <v-card-title class="d-flex align-center justify-space-between">
        <span>Processed PCAPs</span>
        <div class="d-flex ga-2">
          <v-btn :loading="refreshing" density="comfortable" color="secondary" @click="refreshPcaps">
            <v-icon start>mdi-refresh</v-icon>
            Refresh
          </v-btn>
          <v-btn color="primary" density="comfortable" @click="openUploadDialog">
            <v-icon start>mdi-upload</v-icon>
            Upload PCAP
          </v-btn>
        </div>
      </v-card-title>
    </v-card>
    <v-data-table
      :headers="headers"
      :items="store.pcaps || []"
      :loading="store.pcaps === null"
      :items-per-page="20"
      :items-per-page-options="[20, 50, 100, -1]"
      hover
      density="compact"
    >
      <template #[`item.download`]="{ item }"
        ><v-btn
          variant="plain"
          density="compact"
          :href="`/api/download/pcap/${item.Filename}`"
          icon
        >
          <v-icon>mdi-download</v-icon>
        </v-btn></template
      >
      <!-- eslint-disable vue/no-v-for-template-key-on-child -->
      <template
        v-for="field of [
          'ParseTime',
          'PacketTimestampMin',
          'PacketTimestampMax',
        ]"
        #[`item.${field}`]="{ index, value }"
        ><span
          :key="`${field}/${index}`"
          :title="formatDateLong(new Date(value))"
          >{{ formatDate(new Date(value)) }}</span
        ></template
      >
      <template #[`item.Filesize`]="{ value }"
        ><span :title="`${value} Bytes`">{{
          prettyBytes(value, { maximumFractionDigits: 1, binary: true })
        }}</span></template
      >
    </v-data-table>

    <v-dialog v-model="uploadDialog" max-width="600">
      <v-card>
        <v-card-title>Upload PCAP</v-card-title>
        <v-card-text>
          <v-alert
            v-if="uploadError"
            type="error"
            variant="tonal"
            class="mb-4"
            :text="uploadError"
          />
          <v-file-input
            v-model="selectedFile"
            label="Select .pcap or .pcapng file"
            accept=".pcap,.pcapng,application/vnd.tcpdump.pcap,application/octet-stream"
            :disabled="uploading"
            show-size
            density="comfortable"
            prepend-icon="mdi-file"
            @change="onFileChange"
          />
          <v-text-field
            v-model="targetFilename"
            label="Target filename (.pcap or .pcapng)"
            :disabled="uploading"
            density="comfortable"
            clearable
          />
          <v-expand-transition>
            <div v-if="showPasswordField">
              <v-text-field
                v-model="pcapPassword"
                label="PCAP upload password (optional)"
                type="password"
                :disabled="uploading"
                density="comfortable"
                prepend-inner-icon="mdi-lock"
              />
            </div>
          </v-expand-transition>
          <v-progress-linear
            v-if="uploading"
            :model-value="progress"
            height="6"
            color="primary"
            class="mt-2"
            striped
            rounded
          />
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn :disabled="uploading" variant="text" @click="closeUploadDialog">Cancel</v-btn>
          <v-btn :disabled="!canUpload" :loading="uploading" color="primary" @click="startUpload">Upload</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script lang="ts" setup>
import { onMounted, ref, computed } from "vue";
import { useRootStore } from "@/stores";
import { EventBus } from "./EventBus";
import { formatDate, formatDateLong } from "@/filters";
import prettyBytes from "pretty-bytes";
import APIClient from "@/apiClient";

const store = useRootStore();
const headers = [
  {
    title: "File Name",
    value: "Filename",
  },
  {
    title: "First Packet Time",
    value: "PacketTimestampMin",
  },
  {
    title: "Last Packet Time",
    value: "PacketTimestampMax",
  },
  {
    title: "Packet Count",
    value: "PacketCount",
  },
  {
    title: "File Size",
    value: "Filesize",
  },
  {
    title: "Parse Time",
    value: "ParseTime",
    align: "end",
    class: "pr-0",
    cellClass: "pr-0",
  },
  {
    title: "",
    value: "download",
    sortable: false,
    class: ["px-0", "w0"],
    cellClass: ["px-0", "w0"],
  },
] as const;

onMounted(() => {
  store.updatePcaps().catch((err: Error) => {
    EventBus.emit("showError", `Failed to update pcaps: ${err.message}`);
  });
});

const uploadDialog = ref(false);
const refreshing = ref(false);
const selectedFile = ref<File | null>(null);
const targetFilename = ref<string>("");
const pcapPassword = ref<string>("");
const showPasswordField = ref(false);
const uploading = ref(false);
const progress = ref(0);
const uploadError = ref<string | null>(null);

const canUpload = computed(() => {
  if (!selectedFile.value) return false;
  const name = targetFilename.value.trim() || selectedFile.value.name;
  return /\.(pcap|pcapng)$/i.test(name);
});

function openUploadDialog() {
  uploadDialog.value = true;
  uploadError.value = null;
  progress.value = 0;
}

function closeUploadDialog() {
  if (uploading.value) return;
  uploadDialog.value = false;
  selectedFile.value = null;
  targetFilename.value = "";
  pcapPassword.value = "";
  progress.value = 0;
  uploadError.value = null;
}

function onFileChange() {
  if (!selectedFile.value) return;
  // Pre-fill target filename if empty
  if (!targetFilename.value) targetFilename.value = selectedFile.value.name;
}

async function startUpload() {
  if (!selectedFile.value) return;
  const filename = (targetFilename.value || selectedFile.value.name).trim();
  if (!/\.(pcap|pcapng)$/i.test(filename)) {
    uploadError.value = "Filename must end with .pcap or .pcapng";
    return;
  }
  uploading.value = true;
  uploadError.value = null;
  progress.value = 0;
  try {
    await APIClient.uploadPcap(
      selectedFile.value,
      filename,
      pcapPassword.value || undefined,
      (p) => (progress.value = p),
    );
    EventBus.emit("showMessage", `Uploaded ${filename} successfully.`);
    closeUploadDialog();
    await store.updatePcaps();
  } catch (e: unknown) {
    const message =
      typeof e === "object" && e !== null && "message" in e
        ? String((e as { message: unknown }).message)
        : String(e);
    uploadError.value = message;
    // If unauthorized and password field not shown yet, reveal it for retry
    if (/401/.test(message) || /Unauthorized/i.test(message)) {
      showPasswordField.value = true;
    }
    EventBus.emit("showError", `Failed to upload pcap: ${uploadError.value}`);
  } finally {
    uploading.value = false;
  }
}

async function refreshPcaps() {
  if (refreshing.value) return;
  refreshing.value = true;
  try {
    await store.updatePcaps();
  } catch (err: unknown) {
    const message =
      typeof err === "object" && err !== null && "message" in err
        ? String((err as { message: unknown }).message)
        : String(err);
    EventBus.emit("showError", `Failed to update pcaps: ${message}`);
  } finally {
    refreshing.value = false;
  }
}
</script>

<style scoped>
.w0 {
  width: 0;
}
</style>
