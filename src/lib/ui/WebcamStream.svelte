<script lang="ts">
    import { onMount, onDestroy } from 'svelte';

    let videoElement: HTMLVideoElement;
    let stream: MediaStream | null = null;

    async function startWebcam() {
        try {
            stream = await navigator.mediaDevices.getUserMedia({ 
                video: { 
                    width: { ideal: 1280 },
                    height: { ideal: 720 }
                }
            });
            if (videoElement) {
                videoElement.srcObject = stream;
            }
        } catch (err) {
            console.error('Error accessing webcam:', err);
        }
    }

    onMount(() => {
        startWebcam();
    });

    onDestroy(() => {
        if (stream) {
            stream.getTracks().forEach(track => track.stop());
        }
    });
</script>

<div class="webcam-container">
    <video
        bind:this={videoElement}
        autoplay
        playsinline
        muted
        class="rounded-lg"
    />
</div>

<style>
    .webcam-container {
        width: 100%;
        aspect-ratio: 16/9;
        overflow: hidden;
        background: rgba(0, 0, 0, 0.2);
        border-radius: 0.5rem;
    }

    video {
        width: 100%;
        height: 100%;
        object-fit: cover;
    }
</style> 