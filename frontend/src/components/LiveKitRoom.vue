<script setup lang="ts">
import {
	RemoteParticipant,
	RemoteTrack,
	RemoteTrackPublication,
	Room,
	RoomEvent,
	VideoPresets,
	Track,
} from 'livekit-client'
import { onUnmounted, ref } from 'vue'

let _room: Room | undefined
const connected = ref(false)

const state = {
	get room() {
		return _room
	},
	get connected() {
		return connected.value
	},
}

async function joinRoom() {
	const token =
		'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3NDg5NTgxNTcsImlzcyI6IkFQSXdaRURqbWhQZloyViIsIm5iZiI6MTc0ODA1ODE1Nywic3ViIjoiMSIsInZpZGVvIjp7ImNhblB1Ymxpc2giOnRydWUsImNhblB1Ymxpc2hEYXRhIjp0cnVlLCJjYW5TdWJzY3JpYmUiOnRydWUsInJvb20iOiJuaGFuIiwicm9vbUpvaW4iOnRydWV9fQ.2bGjXc3OUQNaoYMG6AZzFGPbsRI7sRr_8oNpX5Y69M8'
	const LIVEKIT_URL = 'wss://ai-service-center-oe0r2q4a.livekit.cloud'

	_room = new Room({
		adaptiveStream: true,
		dynacast: true,
		videoCaptureDefaults: {
			resolution: VideoPresets.h720.resolution,
		},
	})

	_room.on(RoomEvent.ParticipantConnected, (participant) => {
		console.log('✅ Participant connected:', participant.identity)
	})

	_room.on(
		RoomEvent.TrackSubscribed,
		(
			_track: RemoteTrack,
			publication: RemoteTrackPublication,
			participant: RemoteParticipant,
		) => {
			console.log('🔈 Track subscribed:', _track.kind, participant.identity)
			if (_track.kind === Track.Kind.Audio) {
				const audioElement = _track.attach()
				console.log('audioElement', audioElement)
				audioElement.autoplay = true
				audioElement.controls = false
				audioElement.muted = false
				document.body.appendChild(audioElement)
				console.log(`🔊 Đã phát audio từ participant: ${participant.identity}`)
			}
		},
	)

	try {
		await _room.connect(LIVEKIT_URL, token)
		await _room.localParticipant.enableCameraAndMicrophone()
		connected.value = true
		console.log('Participants:', _room.numParticipants)
		console.log('Connected to room', _room.name)
	} catch (error: any) {
		console.error('Connection error:', error.message)
		await leaveRoom()
	}
}

async function leaveRoom() {
	await _room?.disconnect()
	_room = undefined
	connected.value = false
	window.removeEventListener('beforeunload', leaveRoom)
}

onUnmounted(() => {
	leaveRoom()
})
</script>

<template>
	<div id="chat-widget">
		<button
			v-if="!state.connected"
			id="chat-toggle"
			title="Call"
			aria-label="Call"
			@click="joinRoom"
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				width="24"
				height="24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
				class="icon"
				viewBox="0 0 24 24"
			>
				<path d="M12 1v11"></path>
				<path d="M8 5a4 4 0 0 0 8 0"></path>
				<path d="M19 11a7 7 0 0 1-14 0"></path>
				<line x1="12" y1="19" x2="12" y2="23"></line>
				<line x1="8" y1="23" x2="16" y2="23"></line>
			</svg>
		</button>

		<button
			v-else
			id="chat-toggle-disconnect"
			title="Disconnect"
			aria-label="Disconnect"
			@click="leaveRoom"
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				width="24"
				height="24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
				class="icon"
				viewBox="0 0 24 24"
			>
				<line x1="18" y1="6" x2="6" y2="18"></line>
				<line x1="6" y1="6" x2="18" y2="18"></line>
			</svg>
		</button>
	</div>
</template>

<style scoped>
#chat-widget {
	position: fixed;
	bottom: 24px;
	right: 24px;
	z-index: 9999;
	user-select: none;
	font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* Nút gọi - màu xanh dịu */
#chat-toggle {
	width: 56px;
	height: 56px;
	border-radius: 50%;
	border: none;
	background-color: #4a90e2;
	color: white;
	cursor: pointer;
	box-shadow: 0 4px 10px rgba(74, 144, 226, 0.5);
	display: flex;
	align-items: center;
	justify-content: center;
	transition:
		box-shadow 0.3s ease,
		transform 0.15s ease;
}

#chat-toggle:hover {
	box-shadow: 0 6px 20px rgba(74, 144, 226, 0.7);
	transform: scale(1.1);
}

/* Nút ngắt kết nối - màu xám dịu */
#chat-toggle-disconnect {
	width: 56px;
	height: 56px;
	border-radius: 50%;
	border: none;
	background-color: #7f8c8d;
	color: white;
	cursor: pointer;
	box-shadow: 0 4px 10px rgba(127, 140, 141, 0.5);
	display: flex;
	align-items: center;
	justify-content: center;
	transition:
		box-shadow 0.3s ease,
		transform 0.15s ease;
}

#chat-toggle-disconnect:hover {
	box-shadow: 0 6px 20px rgba(127, 140, 141, 0.7);
	transform: scale(1.1);
}

.icon {
	width: 24px;
	height: 24px;
}
</style>
