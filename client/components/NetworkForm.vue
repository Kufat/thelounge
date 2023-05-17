<template>
	<div id="connect" class="window" role="tabpanel" aria-label="Connect">
		<div class="header">
			<SidebarToggle />
		</div>
		<form class="container" method="post" action="" @submit.prevent="onSubmit">
			<h1 class="title">
				<template v-if="defaults.uuid">
					<input v-model="defaults.uuid" type="hidden" name="uuid" />
					Edit {{ defaults.name }}
				</template>
				<template v-else>
					Connect
					<template v-if="config?.lockNetwork"> to {{ defaults.name }} </template>
				</template>
			</h1>

			<h2>User preferences</h2>
			<div class="connect-row">
				<label for="connect:nick"
					>Registered nickname (no spaces or special characters) e.g. DrExample</label
				>
				<input
					id="connect:nick"
					v-model="defaults.nick"
					class="input nick"
					name="nick"
					pattern="[^\s:!@]+"
					maxlength="100"
					required
					@input="onNickChanged"
				/>
			</div>
			<div class="connect-row">
				<label for="connect:realname"
					>WHOIS name/identity (spaces and special characters allowed, shown in /whois)
					e.g. Dr. Jamie Example</label
				>
				<input
					id="connect:realname"
					v-model.trim="defaults.realname"
					class="input"
					name="realname"
					maxlength="300"
				/>
			</div>
			<div class="connect-row">
				<label for="connect:leaveMessage"
					>Leave message (displayed when you disconnect)</label
				>
				<input
					id="connect:leaveMessage"
					v-model.trim="defaults.leaveMessage"
					autocomplete="off"
					class="input"
					name="leaveMessage"
					placeholder="The Lounge - https://thelounge.chat"
				/>
			</div>
			<template v-if="defaults.uuid && !store.state.serverConfiguration?.public">
				<div class="connect-row">
					<label for="connect:commands">
						Commands
						<span
							class="tooltipped tooltipped-ne tooltipped-no-delay"
							aria-label="One /command per line.
Each command will be executed in
the server tab on new connection"
						>
							<button class="extra-help" />
						</span>
					</label>
					<textarea
						id="connect:commands"
						ref="commandsInput"
						autocomplete="off"
						:value="defaults.commands ? defaults.commands.join('\n') : ''"
						class="input"
						name="commands"
						@input="resizeCommandsInput"
					/>
				</div>
			</template>
			<template v-else-if="!defaults.uuid">
				<div class="connect-row">
					<label for="connect:channels">Channels</label>
					<input
						id="connect:channels"
						v-model.trim="defaults.join"
						class="input"
						name="join"
					/>
				</div>
			</template>

			<template v-if="store.state.serverConfiguration?.public">
				<template v-if="config?.lockNetwork">
					<div class="connect-row">
						<label></label>
						<div class="input-wrap">
							<label class="tls">
								<input v-model="displayPasswordField" type="checkbox" />
								I have a password
							</label>
						</div>
					</div>
					<div v-if="displayPasswordField" class="connect-row">
						<label for="connect:password">Password</label>
						<RevealPassword
							v-slot:default="slotProps"
							class="input-wrap password-container"
						>
							<input
								id="connect:password"
								ref="publicPassword"
								v-model="defaults.password"
								class="input"
								:type="slotProps.isVisible ? 'text' : 'password'"
								placeholder="Server password (optional)"
								name="password"
								maxlength="300"
							/>
						</RevealPassword>
					</div>
				</template>
			</template>
			<template v-else>
				<h2 id="label-auth">Authentication</h2>
				<div class="connect-row connect-auth" role="group" aria-labelledby="label-auth">
					<label class="opt">
						<input
							:checked="!defaults.sasl"
							type="radio"
							name="sasl"
							value=""
							@change="setSaslAuth('')"
						/>
						No authentication
					</label>
					<label class="opt">
						<input
							:checked="defaults.sasl === 'plain'"
							type="radio"
							name="sasl"
							value="plain"
							@change="setSaslAuth('plain')"
						/>
						Username + password (SASL PLAIN)
					</label>
					<label
						v-if="!store.state.serverConfiguration?.public && defaults.tls"
						class="opt"
					>
						<input
							:checked="defaults.sasl === 'external'"
							type="radio"
							name="sasl"
							value="external"
							@change="setSaslAuth('external')"
						/>
						Client certificate (SASL EXTERNAL)
					</label>
				</div>

				<template v-if="defaults.sasl === 'plain'">
					<div class="connect-row">
						<label for="connect:username">Account</label>
						<input
							id="connect:saslAccount"
							v-model.trim="defaults.saslAccount"
							class="input"
							name="saslAccount"
							maxlength="100"
							required
						/>
					</div>
					<div class="connect-row">
						<label for="connect:password">Password</label>
						<RevealPassword
							v-slot:default="slotProps"
							class="input-wrap password-container"
						>
							<input
								id="connect:saslPassword"
								v-model="defaults.saslPassword"
								class="input"
								:type="slotProps.isVisible ? 'text' : 'password'"
								name="saslPassword"
								maxlength="300"
								required
							/>
						</RevealPassword>
					</div>
				</template>
				<div v-else-if="defaults.sasl === 'external'" class="connect-sasl-external">
					<p>The Lounge automatically generates and manages the client certificate.</p>
					<p>
						On the IRC server, you will need to tell the services to attach the
						certificate fingerprint (certfp) to your account, for example:
					</p>
					<pre><code>/msg NickServ CERT ADD</code></pre>
				</div>
			</template>

			<div>
				<button type="submit" class="btn" :disabled="disabled ? true : false">
					<template v-if="defaults.uuid">Save network</template>
					<template v-else>Connect</template>
				</button>
			</div>
		</form>
		<p>
			<a href="https://skipirc.miraheze.org/wiki/Main_Page">SkipIRC rules and information</a>
		</p>
		<p><a href="https://scp-wiki.wikidot.com/chat-guide">SCP Wiki chat guide</a></p>
		<p>
			If you run into any difficulties, please
			<a href="mailto:kufat@kufat.net">contact Kufat</a>. Thanks for using SkipIRC!
		</p>
	</div>
</template>

<style>
#connect .connect-auth {
	display: block;
	margin-bottom: 10px;
}

#connect .connect-auth .opt {
	display: block;
	width: 100%;
}

#connect .connect-auth input {
	margin: 3px 10px 0 0;
}

#connect .connect-sasl-external {
	padding: 10px;
	border-radius: 2px;
	background-color: #d9edf7;
	color: #31708f;
}

#connect .connect-sasl-external pre {
	margin: 0;
	user-select: text;
}
</style>

<script lang="ts">
import RevealPassword from "./RevealPassword.vue";
import SidebarToggle from "./SidebarToggle.vue";
import {defineComponent, nextTick, PropType, ref, watch} from "vue";
import {useStore} from "../js/store";
import {ClientNetwork} from "../js/types";

export type NetworkFormDefaults = Partial<ClientNetwork> & {
	join?: string;
};

export default defineComponent({
	name: "NetworkForm",
	components: {
		RevealPassword,
		SidebarToggle,
	},
	props: {
		handleSubmit: {
			type: Function as PropType<(network: ClientNetwork) => void>,
			required: true,
		},
		defaults: {
			type: Object as PropType<NetworkFormDefaults>,
			required: true,
		},
		disabled: Boolean,
	},
	setup(props) {
		const store = useStore();
		const config = ref(store.state.serverConfiguration);
		const previousUsername = ref(props.defaults?.username);
		const displayPasswordField = ref(false);

		const publicPassword = ref<HTMLInputElement | null>(null);

		watch(displayPasswordField, (newValue) => {
			if (newValue) {
				void nextTick(() => {
					publicPassword.value?.focus();
				});
			}
		});

		const commandsInput = ref<HTMLInputElement | null>(null);

		const resizeCommandsInput = () => {
			if (!commandsInput.value) {
				return;
			}

			// Reset height first so it can down size
			commandsInput.value.style.height = "";

			// 2 pixels to account for the border
			commandsInput.value.style.height = `${Math.ceil(
				commandsInput.value.scrollHeight + 2
			)}px`;
		};

		watch(
			() => props.defaults?.commands,
			() => {
				void nextTick(() => {
					resizeCommandsInput();
				});
			}
		);

		watch(
			() => props.defaults?.tls,
			(isSecureChecked) => {
				const ports = [6667, 6697];
				const newPort = isSecureChecked ? 0 : 1;

				// If you disable TLS and current port is 6697,
				// set it to 6667, and vice versa
				if (props.defaults?.port === ports[newPort]) {
					props.defaults.port = ports[1 - newPort];
				}
			}
		);

		const setSaslAuth = (type: string) => {
			if (props.defaults) {
				props.defaults.sasl = type;
			}
		};

		const usernameInput = ref<HTMLInputElement | null>(null);

		const onNickChanged = (event: Event) => {
			if (!usernameInput.value) {
				return;
			}

			const usernameRef = usernameInput.value;

			if (!usernameRef.value || usernameRef.value === previousUsername.value) {
				usernameRef.value = (event.target as HTMLInputElement)?.value;
			}

			previousUsername.value = (event.target as HTMLInputElement)?.value;
		};

		const onSubmit = (event: Event) => {
			const formData = new FormData(event.target as HTMLFormElement);
			const data: Partial<ClientNetwork> = {};

			formData.forEach((value, key) => {
				data[key] = value;
			});

			props.handleSubmit(data as ClientNetwork);
		};

		return {
			store,
			config,
			displayPasswordField,
			publicPassword,
			commandsInput,
			resizeCommandsInput,
			setSaslAuth,
			usernameInput,
			onNickChanged,
			onSubmit,
		};
	},
});
</script>
