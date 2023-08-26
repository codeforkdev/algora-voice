<script lang="ts">
	// import { browser } from '$app/environment';
	import { invalidateAll } from '$app/navigation';

	import type {
		IAgoraRTC,
		IAgoraRTCClient,
		IAgoraRTCRemoteUser,
		IMicrophoneAudioTrack,
		IRemoteAudioTrack
	} from 'agora-rtc-sdk-ng';
	import { onMount } from 'svelte';
	let users: IAgoraRTCRemoteUser[] = [];
	let appId: string = '1eb79e166f7243de8a37514b86d967b0';
	let channel: string = 'test';
	let token: string =
		'007eJxTYHi3n3ninSWl02M+uPEUNybMU0u8VGbg/jAovf7f7QnailoKDIapSeaWqYZmZmnmRibGKakWicbmpoYmSRZmKZZm5kkG29pepjQEMjIIOV5lZmSAQBCfhaEktbiEgQEAWOUfbg==';
	let uid: string;

	let channelParams = {
		// @ts-ignore
		localAudioTrack: null as IMicrophoneAudioTrack,
		// @ts-ignore
		remoteAudioTrack: null as IRemoteAudioTrack,
		remoteUid: Math.random().toString() as number | string
	};
	let agoraRTC: IAgoraRTC;
	let agoraEngine: IAgoraRTCClient;

	onMount(async () => {
		agoraRTC = (await import('agora-rtc-sdk-ng')).default;
		agoraEngine = agoraRTC.createClient({ mode: 'rtc', codec: 'vp9' });
		async function startBasicCall() {
			agoraEngine.on('user-published', async (user, mediaType) => {
				console.log('subscribe success*******************************************************');
				await agoraEngine.subscribe(user, mediaType);

				users = [...users, user];

				if (mediaType == 'audio') {
					channelParams.remoteUid = user.uid;
					//@ts-ignore
					channelParams.remoteAudioTrack = user.audioTrack;
					channelParams.remoteAudioTrack.play();
					console.log('Remote user connected');
				}

				agoraEngine.on('user-unpublished', (user) => {
					console.log(
						user.uid,
						'has left the channel *************************************************'
					);
					users = users.filter((u) => u.uid !== user.uid);
				});
			});
		}
		await startBasicCall();
	});

	async function join() {
		await agoraEngine.join(appId, channel, token, uid);
		console.log('Joined channel:', channel);

		channelParams.localAudioTrack = await agoraRTC.createMicrophoneAudioTrack();

		await agoraEngine.publish(channelParams.localAudioTrack);
		console.log('Publish success!****************************************');
	}

	async function leave() {
		channelParams.localAudioTrack.close();

		await agoraEngine.leave();
		console.log('You left the channel');
		// await invalidateAll();
	}
</script>

<h2 class="">Get started with Voice Calling</h2>
<div class="row">
	<div>
		<button on:click={join} type="button" id="join" class="bg-blue-500 py-1 px-4 rounded-lg"
			>Join</button
		>
		<button on:click={leave} type="button" id="leave" class="bg-red-500 py-1 px-4 rounded-lg"
			>Leave</button
		>
	</div>
</div>
<br />

<div>
	{#each users as user}{/each}
</div>
<div id="message" />
