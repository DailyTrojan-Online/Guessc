<script lang="ts">
    import data from "$lib/assets/image.json";
    let showModal = false;
    import { onMount } from "svelte";
    let randint = Math.floor(Math.random() * data.images.length);
    let canvas, ctx;
    let canvas2 , ctx2;
    let mousex , mousey;
    let mapRect;
    let map;
    let clickx, clicky;
    let dist;
    let positionx;
    let positiony;
    let Imagepath;
    onMount(async () => {
        if(data.images && data.images.length > 0){
            positionx = data.images[randint]["position-x"];
            positiony = data.images[randint]["position-y"];
            Imagepath = data.images[randint]["path"];
        }
        mapRect = map.getBoundingClientRect();
        if(canvas && map){
            console.log("Canvas", canvas);
            ctx = canvas.getContext("2d");
            resizeCanvas();
        }
        if(canvas2){
            console.log("Canvas2", canvas2);
            ctx2 = canvas2.getContext("2d");
            resizeCanvas();
        }
        window.addEventListener("resize", resizeCanvas);
    } )
    $: if(map != undefined) {
        mapRect = map.getBoundingClientRect();
    }
    function mouseMove(event) {
        mapRect = map.getBoundingClientRect();
        mousex = (event.clientX - mapRect.left) / mapRect.width;
        mousey = (event.clientY - mapRect.top) / mapRect.height;
        if (mousex < 0 || mousey < 0) {
            mousex = 0;
            mousey = 0;
        }
    }
    function mouseclick(event) {
        clickx = mousex;
        clicky = mousey;
        
        dist = Math.sqrt((clickx-positionx) ** 2 + (clicky-positiony) ** 2);
        dist = dist.toFixed(2);
        dist = 5000-dist * 5000;
        drawMarker(clickx, clicky);
    }
    function resizeCanvas() {
        if (canvas && map && mapRect) {
            canvas.width = mapRect.width;
            canvas.height = mapRect.height;
            // Redraw markers or elements on the canvas after resizing
        }
        if(canvas2 && mapRect){
            console.log("Resizing canvas2");
            canvas2.width = mapRect.width;
            canvas2.height = mapRect.height;
        }

    } 
    function drawMarker(x, y) {
        
        if (ctx && canvas) {
            ctx.clearRect(0, 0, canvas.width, canvas.height); // Clear previous drawings

            // Draw a marker at the specified position (scaled to canvas dimensions)
            const markerX = x * canvas.width;
            const markerY = y * canvas.height;
            ctx.beginPath();
            ctx.arc(markerX, markerY, 5, 0, 2 * Math.PI); // Draw a circle marker
            ctx.fillStyle = "red"; // Marker color
            ctx.fill();
        }
        if(ctx2 && canvas2){
            console.log("Drawing modal marker at", x, y);
            ctx2.clearRect(0, 0, canvas2.width, canvas2.height);
            const markerX2 = x * canvas2.width;
            const markerY2 = y * canvas2.height;
            ctx2.beginPath();
            ctx2.arc(markerX2, markerY2, 5, 0, 2 * Math.PI); // Draw a circle marker
            ctx2.fillStyle = "red"; // Marker color
            ctx2.fill();
        }
    }
    function buttonclick(event){
        showModal = true;
    }
</script>
<svelte:window on:pointermove={mouseMove}></svelte:window>
<div class="game-wrapper">
    <!-- <h1>Coordinates</h1>
    <p>X: {xfixed} y: {yfixed}</p>
    <p>X: {mousex} y: {mousey}</p>
    <p>clickX: {clickx} clickY: {clicky}</p> -->

    <div class="image">
        <img src={Imagepath} alt="Guess-Image" />
    </div>
    <div class="map-wrapper">
        <div class="map">
            <div bind:this={map} on:click={mouseclick} class="image-wrapper">
                <img src="./Map.png" alt="Map" />
                <canvas bind:this={canvas} class="map-canvas"></canvas>
            </div>
            <button on:click={buttonclick}>Submit</button>
        </div>
    </div>
</div>
{#if showModal}
    <div class="modal">
        <div class="modal-content">
            <div class = "modal-map-wrapper">
                <img src="./Map.png" alt="Map" />
                <canvas bind:this={canvas2} class="modal-canvas"></canvas>
            </div>
            <p>Distance: {dist}</p>
            <button on:click={() => showModal = false}>Close</button>
        </div>
    </div>
{/if}
<style>
    .modal-content img{
        width: 100%;

    }
    .modal{
        position: fixed;
        top:50%;
        left:50%;
        transform: translate(-50%, -50%);
        width: 600px;
        z-index: 999;
        background-color: grey;
    }
    .game-wrapper {
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        box-sizing: border-box;
    }
    .x-axis {
        width: 1px;
        height: 1px;
        background-color: red;
    }
    .y-axis {
        width: 1px;
        height: 1px;
        background-color: blue;
    }
    .image {
        width: 100%;
        height: 100%;
        border: 1px solid black;
        box-sizing: border-box;
    }
    .image img {
        width: 100%;
        height: 100%;
        position: absolute;
    }
    .map-wrapper {
        position: relative;
        border: 1px solid black;
    }
    .map {
        position: fixed;
        bottom: 0;
        right: 0;
        border: 1px solid black;
        background-color: white;
    }
    .map img {
        width: 300px;
    }
    .image-wrapper {
        transition: 0.2s;
        transform-origin: bottom right;
        position: relative;
    }
    .image-wrapper:hover {
        cursor: crosshair;
        transform: scale(1.5);
    }
    .image-wrapper canvas {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }
    .modal-map-wrapper canvas{
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }
</style>
