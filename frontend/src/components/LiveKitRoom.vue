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
import { onUnmounted } from 'vue'

let _room: Room | undefined

const state = {
	get room() {
		return _room
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

	_room.on(RoomEvent.TrackSubscribed, (track, pub, participant) => {
		console.log('🔈 Track subscribed:', track.kind, participant.identity)
	})

	try {
		await _room.connect(LIVEKIT_URL, token)
		await _room.localParticipant.enableCameraAndMicrophone()
		console.log('Participants:', _room.numParticipants)
		console.log('Connected to room', _room.name)
	} catch (error: any) {
		console.error('Connection error:', error.message)
		await leaveRoom()
	}
}

async function leaveRoom() {
	await _room?.disconnect()

	// Empty all variables
	_room = undefined

	window.removeEventListener('beforeunload', leaveRoom)
}

onUnmounted(() => {
	leaveRoom()
})
</script>

<template>
	<div v-if="!state.room" id="join">
		<div id="join-dialog" @click="joinRoom">
			<button title="Call">📞</button>
		</div>
	</div>
	<div v-else id="room">
		<div id="room-header">
			<button title="Disconnect" @click="leaveRoom">❌</button>
		</div>
	</div>
</template>

<style scoped>
#chat-widget {
	position: fixed;
	bottom: 24px;
	right: 24px;
	z-index: 9999;
	font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
	user-select: none;
}

/* Nút icon tròn khi chưa mở */
#chat-toggle {
	width: 56px;
	height: 56px;
	border-radius: 50%;
	background: linear-gradient(135deg, #10a37f, #22c55e);
	box-shadow: 0 8px 15px rgba(16, 163, 127, 0.4);
	color: white;
	font-size: 28px;
	border: none;
	cursor: pointer;
	display: flex;
	align-items: center;
	justify-content: center;
	transition: box-shadow 0.3s ease;
}

#chat-toggle:hover {
	box-shadow: 0 12px 20px rgba(16, 163, 127, 0.7);
}

/* Khung chat mở rộng */
#chat-window {
	width: 320px;
	height: 420px;
	background: white;
	border-radius: 12px;
	box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
	display: flex;
	flex-direction: column;
	overflow: hidden;
	font-size: 14px;
	color: #222;
}

/* Header khung chat */
#chat-header {
	background: linear-gradient(135deg, #10a37f, #22c55e);
	color: white;
	padding: 12px 16px;
	font-weight: 600;
	display: flex;
	align-items: center;
	justify-content: space-between;
	font-size: 16px;
}

/* Nút đóng */
#chat-close-btn {
	background: transparent;
	border: none;
	color: white;
	font-size: 22px;
	cursor: pointer;
	line-height: 1;
}

/* Phần nội dung chat - tạm để trống */
#chat-content {
	flex-grow: 1;
	padding: 16px;
	overflow-y: auto;
	background: #f9f9f9;
}

/* Ẩn khung chat mặc định */
#chat-window.hidden {
	display: none;
}
</style>
