<script setup lang="ts">
import {
	LocalVideoTrack,
	RemoteParticipant,
	RemoteTrack,
	RemoteTrackPublication,
	Room,
	RoomEvent,
	VideoPresets,
	Track,
} from 'livekit-client'
import { onUnmounted } from 'vue'

type TrackInfo = {
	trackPublication: RemoteTrackPublication
	participantIdentity: string
}

let _room: Room | undefined

const state = {
	get room() {
		return _room
	},
}

async function joinRoom() {
	const token =
		'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3NDc2NTM0NjcsImlzcyI6IkFQSWo4b2U4VWdncjVKRSIsIm5iZiI6MTc0NzU2MzQ2Nywic3ViIjoiTmhhbiIsInZpZGVvIjp7ImNhblB1Ymxpc2giOnRydWUsImNhblB1Ymxpc2hEYXRhIjp0cnVlLCJjYW5TdWJzY3JpYmUiOnRydWUsInJvb20iOiJyb29tMSIsInJvb21Kb2luIjp0cnVlfX0.Vr1PkvoELdP2MZDRoVqb4ULaP5BKbsH-4W7K4iqPOcU'
	const LIVEKIT_URL = 'wss://ai-service-center-oe0r2q4a.livekit.cloud'

	_room = new Room({
		adaptiveStream: true,
		dynacast: true,
		videoCaptureDefaults: {
			resolution: VideoPresets.h720.resolution,
		},
	})

	_room.prepareConnection(LIVEKIT_URL, token)

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

	_room.on(
		RoomEvent.TrackUnsubscribed,
		(_track: RemoteTrack, publication: RemoteTrackPublication) => {
			_remoteTracksMap.delete(publication.trackSid)
		},
	)

	try {
		await _room.connect(LIVEKIT_URL, token)
		await _room.localParticipant.enableCameraAndMicrophone()
		console.log('coneected to room', _room.name)
	} catch (error: any) {
		console.error('Connection error:', error.message)
		await leaveRoom()
	}

	window.addEventListener('beforeunload', leaveRoom)
}

async function leaveRoom() {
	await _room.disconnect()

	// Empty all variables
	_room = undefined
	_localTrack = undefined

	window.removeEventListener('beforeunload', leaveRoom)
}

onUnmounted(() => {
	leaveRoom()
})
</script>

<template>
	<div v-if="!state.room" id="join">
		<div id="join-dialog">
			<h2>Assistant</h2>
			<form @submit.prevent="joinRoom">
				<button class="btn btn-lg btn-success" type="submit">Call</button>
			</form>
		</div>
	</div>
	<div v-else id="room">
		<div id="room-header">
			<button class="btn btn-danger" id="leave-room-button" @click="leaveRoom">
				Disconnect
			</button>
		</div>
	</div>
</template>
