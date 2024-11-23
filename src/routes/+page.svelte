<script lang="ts">
    import data from "$lib/assets/image.json";
    let showModal = false;
    import { onMount } from "svelte";
    import { tick } from "svelte";
    let guessmade = false;
    let randint = Math.floor(Math.random() * data.images.length);
    let canvas, ctx;
    let canvas2 , ctx2;
    let mousex , mousey;
    let mapRect, mapRect2;
    let map, map2;
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
        console.log("Click postion x,y : ",clickx, clicky);
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
    } 
    function drawLineBetweenPoints(x1, y1, x2, y2) {
        if (ctx2 && canvas2) {
            ctx2.beginPath();
            ctx2.moveTo(x1 * canvas2.width, y1 * canvas2.height); // Start point of the line
            ctx2.lineTo(x2 * canvas2.width, y2 * canvas2.height); // End point of the line
            ctx2.setLineDash([5, 5]);
            ctx2.strokeStyle = "blue"; // Line color
            ctx2.lineWidth = 2; // Line thickness
            ctx2.stroke(); // Draw the line
            ctx2.setLineDash([]); // Reset the line dash pattern
        }
    }
    function drawFinishMarker(x, y) {
        if (ctx2 && canvas2) {
        const img = new Image(); // Create a new image element
        img.src = "./guesSC.png"; // Set the path to the PNG

        img.onload = function () {
            // Once the image is loaded, draw it on the canvas
            ctx2.drawImage(img, x * canvas2.width-15, y * canvas2.height-15, 30, 30);
        };
    }
    }
    function resizeCanvas2() {
        
        if (canvas2 && map2 && mapRect2) {
            canvas2.width = mapRect2.width;
            canvas2.height = mapRect2.height;
            drawLineBetweenPoints(clickx, clicky, positionx, positiony);
            drawMarker2(clickx, clicky);
            drawFinishMarker(positionx, positiony);
            // Redraw markers or elements on the canvas after resizing
        }
    }
    function drawMarker(x, y) {
        
        if (ctx && canvas) {
            ctx.clearRect(0, 0, canvas.width, canvas.height); // Clear previous drawings
            // Draw a marker at the specified position (scaled to canvas dimensions)
            const markerX = x * canvas.width;
            const markerY = y * canvas.height;
            ctx.beginPath();
            ctx.arc(markerX, markerY, 3, 0, 2 * Math.PI); // Draw a circle marker
            ctx.fillStyle = "red"; // Marker color
            ctx.fill();
        }
    }
    function drawMarker2(x, y) {
        if (ctx2 && canvas2) {// Clear previous drawings
            // Draw a marker at the specified position (scaled to canvas dimensions)
            const markerX = x * canvas2.width;
            const markerY = y * canvas2.height;
            ctx2.beginPath();
            ctx2.arc(markerX, markerY, 5, 0, 2 * Math.PI); // Draw a circle marker
            ctx2.fillStyle = "red"; // Marker color
            ctx2.fill();
        }
    }
    function setupCanvas2(){
        mapRect2 = map2.getBoundingClientRect();
        if(canvas2 && map2){
            console.log("Canvas2", canvas2);
            ctx2 = canvas2.getContext("2d");
            resizeCanvas2();
        }
    }
    function buttonclick(event){
        guessmade = true;
        showModal = true;
        tick().then(setupCanvas2);
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
            <div bind:this={map} on:click={guessmade ? null: mouseclick} class="image-wrapper">
                <img src="./Map.png" alt="Map" />
                <canvas bind:this={canvas} class="map-canvas"></canvas>
            </div>
            <button on:click={guessmade ? null:buttonclick} disabled={guessmade}>Submit</button>
        </div>
    </div>
</div>
{#if showModal}
    <div class="modal">
        <div class="modal-content">
            <div bind:this={map2} class = "modal-map-wrapper">
                <img src="./Map.png" alt="Map" />
                <canvas bind:this={canvas2} class="modal-canvas"></canvas>
            </div>
            <div class="my-4">
                <div class="mb-1 text-lg font-medium dark:text-white">Extra Large</div>
                <Progressbar progress="50" size="h-6" />
            </div>
            <p>Distance: {dist}</p>
            <button on:click={() => showModal = false}>Close</button>
        </div>
    </div>
{/if}
<style>
    .modal-content{
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        align-items: center;
    }
    .modal-map-wrapper{
        position: relative;
        border: 1px solid black;
    }
    .modal-map-wrapper img{
        width:100%;
        height:100%;
        position: relative;
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
        background-color: black;
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
        cursor: crosshair;
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
