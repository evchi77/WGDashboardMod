<script>
import {fetchGet} from "@/utilities/fetch.js";
import {DashboardConfigurationStore} from "@/stores/DashboardConfigurationStore.js";
import LocaleText from "@/components/text/localeText.vue";
import ProtocolBadge from "@/components/protocolBadge.vue";
import ConfigurationToggleConfirm from "@/components/configurationComponents/configurationToggleConfirm.vue";

export default {
	name: "configurationCard",
	components: {ConfigurationToggleConfirm, ProtocolBadge, LocaleText},
	props: {
		c: {
			Name: String,
			Status: Boolean,
			PublicKey: String,
			PrivateKey: String
		},
		delay: String,
		display: String
	},
	data(){
		return{
			configurationToggling: false,
			pendingToggleTarget: undefined,
			confirmToggleOpen: false
		}
	},
	setup(){
		const dashboardConfigurationStore = DashboardConfigurationStore();
		return {dashboardConfigurationStore}
	},
	methods: {
		requestToggle(event){
			if (event){
				event.preventDefault();
			}
			if (this.configurationToggling){
				return;
			}
			if (this.c.Status){
				this.confirmToggleOpen = true;
				return;
			}
			this.toggle();
		},
		confirmToggle(){
			this.confirmToggleOpen = false;
			this.toggle();
		},
		toggle(){
			this.configurationToggling = true;
			this.pendingToggleTarget = !this.c.Status;
			fetchGet("/api/toggleWireguardConfiguration", {
				configurationName: this.c.Name
			}, (res) => {
				if (res.status){
					this.dashboardConfigurationStore.newMessage("Server",
						`${this.c.Name} ${res.data ? 'is on':'is off'}`)
				}else{
					this.dashboardConfigurationStore.newMessage("Server",
						res.message, 'danger')
				}
				this.c.Status = res.data
				this.configurationToggling = false;
				this.pendingToggleTarget = undefined;
			})
		}
	}
}
</script>

<template>
	<div class="col-12" :class="{'col-lg-6 col-xl-4': this.display === 'Grid'}">
		<div class="card conf_card rounded-3 shadow text-decoration-none">
			<RouterLink :to="'/configuration/' + c.Name + '/peers'" class="card-body d-flex align-items-center gap-3 flex-wrap text-decoration-none">
				<h6 class="mb-0"><span class="dot" :class="{active: c.Status}"></span></h6>
				<h6 class="card-title mb-0 d-flex align-items-center gap-2">
					<samp>{{c.Name}}</samp>
					<small>
						<ProtocolBadge :protocol="c.Protocol" :mini="true"></ProtocolBadge>
					</small>
					<small v-if="c.Info.Description">
						<span class="badge text-bg-info rounded-3 shadow">
							<i class="bi bi-pencil-fill me-2"></i>
							{{ c.Info.Description }}
						</span>
					</small>
				</h6>
				<h6 class="mb-0 ms-auto">
					<i class="bi bi-chevron-right"></i>
				</h6>
			</RouterLink>
			<div class="card-footer d-flex gap-2 flex-column">
				<div class="row">
					<small class="col-6" :class="{'col-md-3': this.display === 'List'}">
						<i class="bi bi-arrow-down-up me-2"></i>{{c.DataUsage.Total > 0 ? c.DataUsage.Total.toFixed(4) : 0}} GB
					</small>
					<small class="text-primary-emphasis col-6" :class="{'col-md-3': this.display === 'List'}">
						<i class="bi bi-arrow-down me-2"></i>{{c.DataUsage.Receive > 0 ? c.DataUsage.Receive.toFixed(4) : 0}} GB
					</small>
					<small class="text-success-emphasis col-6" :class="{'col-md-3': this.display === 'List'}">
						<i class="bi bi-arrow-up me-2"></i>{{c.DataUsage.Sent > 0 ? c.DataUsage.Sent.toFixed(4) : 0}} GB
					</small>
					<small class="col-6" :class="{'col-md-3 text-md-end ': this.display === 'List'}">
						<span class="dot me-2" :class="{active: c.ConnectedPeers > 0}"></span>
						{{c.ConnectedPeers}} / {{c.TotalPeers}}
						<LocaleText t="Peers"></LocaleText>
					</small>
				</div>
				<div class="d-flex gap-2 " :class="[this.display === 'Grid' ? 'flex-column': 'gap-lg-3 flex-column flex-lg-row']">
					<div class="d-flex gap-2 align-items-center">
						<small class="text-muted">
							<strong>
								<LocaleText t="Public Key"></LocaleText>
							</strong>
						</small>
						<small class="mb-0 d-block d-lg-inline-block ">
							<samp style="line-break: anywhere">{{c.PublicKey}}</samp>
						</small>
					</div>
					<div class="form-check form-switch ms-auto">
						<label class="form-check-label" style="cursor: pointer" :for="'switch' + c.PrivateKey">
							<LocaleText t="Turning Off..." v-if="this.configurationToggling && this.pendingToggleTarget === false"></LocaleText>
							<LocaleText t="Turning On..." v-else-if="this.configurationToggling && this.pendingToggleTarget === true"></LocaleText>
							<LocaleText t="On" v-else-if="c.Status && !this.configurationToggling"></LocaleText>
							<LocaleText t="Off" v-else-if="!c.Status && !this.configurationToggling"></LocaleText>


							<span v-if="this.configurationToggling"
							      class="spinner-border spinner-border-sm ms-2" aria-hidden="true"></span>
						</label>
						<input class="form-check-input"
						       style="cursor: pointer"
						       :disabled="this.configurationToggling"
						       type="checkbox" role="switch" :id="'switch' + c.PrivateKey"
						       @click="requestToggle"
						       :checked="c.Status"
						>
					</div>
				</div>
			</div>
		</div>
		<Transition name="zoom">
			<ConfigurationToggleConfirm
				v-if="confirmToggleOpen"
				:configurationName="c.Name"
				@confirm="confirmToggle"
				@close="confirmToggleOpen = false"
			></ConfigurationToggleConfirm>
		</Transition>
	</div>
</template>

<style scoped>
.fade-enter-active{
	transition-delay: v-bind(delay) !important;
}
</style>
