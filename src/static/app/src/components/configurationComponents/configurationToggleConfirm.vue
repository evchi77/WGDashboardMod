<script setup>
import LocaleText from "@/components/text/localeText.vue";

const props = defineProps({
	configurationName: {
		type: String,
		required: true
	}
})

const emits = defineEmits(["confirm", "close"])
</script>

<template>
	<div class="peerSettingContainer w-100 h-100 position-absolute top-0 start-0 overflow-y-scroll">
		<div class="container d-flex h-100 w-100">
			<div class="m-auto modal-dialog-centered dashboardModal" style="width: 700px">
				<div class="card rounded-3 shadow flex-grow-1 bg-warning-subtle border-warning-subtle">
					<div class="card-header bg-transparent d-flex align-items-center gap-2 border-0 p-4 pb-0">
						<h5 class="mb-0">
							<LocaleText t="Disable Tunnel"></LocaleText>
						</h5>
						<button type="button" class="btn-close ms-auto" @click="emits('close')"></button>
					</div>
					<div class="card-body px-4 text-muted">
						<p class="mb-2">
							<LocaleText t="Are you sure you want to disable this tunnel?"></LocaleText>
						</p>
						<p class="mb-3">
							<strong><LocaleText t="Configuration"></LocaleText>:</strong>
							<samp>{{ configurationName }}</samp>
						</p>
						<ul class="mb-3">
							<li>
								<LocaleText t="This will disconnect all connected peers."></LocaleText>
							</li>
						</ul>
						<div class="d-flex gap-2">
							<button class="btn btn-outline-secondary" @click="emits('close')">
								<LocaleText t="Cancel"></LocaleText>
							</button>
							<button class="btn btn-danger ms-auto" @click="emits('confirm')">
								<LocaleText t="Disable"></LocaleText>
							</button>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style scoped>
</style>
