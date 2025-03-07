<script lang="ts">
    import data from "$lib/assets/image.json";
    let showModal = false;
    import { onMount } from "svelte";
    import { tick } from "svelte";
    import { blur, fly, slide } from "svelte/transition";
    let guessmade = false;
    let guessed = false;
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
    let progress = 0;
    let progress_color = '#ff9595';
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
        guessed = true;
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
            ctx2.strokeStyle = "#990000"; // Line color
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
            ctx2.drawImage(img, x * canvas2.width-10, y * canvas2.height-10, 20, 20);
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
            ctx.fillStyle = "#990000"; // Marker color
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
            ctx2.fillStyle = "#990000"; // Marker color
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
        tick().then(changeprogress);
    }
    const randomTime = () => Math.floor(Math.random() * 60);
    const changeprogress = () => progress = dist/5000 * 100;
</script>
<svelte:window on:pointermove={mouseMove}></svelte:window>
<div class="game-wrapper" style:filter={guessmade && showModal ? "blur(10px)" : "none"}>
    <!-- <h1>Coordinates</h1>
    <p>X: {xfixed} y: {yfixed}</p>
    <p>X: {mousex} y: {mousey}</p>
    <p>clickX: {clickx} clickY: {clicky}</p> -->

    <div class="image" >
        <img src={Imagepath} alt="Guess-Image" />
    </div>
    {#if !guessmade}
    <div class="map-wrapper" transition:fly={{duration: 500, x: 100}}>
        <div class="map">
            <div bind:this={map} on:click={guessmade ? null: mouseclick}  class="image-wrapper" style:pointer-events={guessmade ? "none" : "auto"}>
                <img src="./Map.png" alt="Map" />
                <canvas bind:this={canvas} class="map-canvas"></canvas>
            </div>
            <div >
            {#if !guessmade && guessed}
            <button  transition:fly={{duration: 300, y: 100}} on:click={guessmade ? null:buttonclick} class="button-guess">Submit</button>
            {/if}</div>
        </div>
    </div>
    {/if}
</div>
{#if showModal}
    <div class="modal">
        <div class="modal-content">
            <div bind:this={map2} class = "modal-map-wrapper">
                <img src="./Map.png" alt="Map" />
                <canvas bind:this={canvas2} class="modal-canvas"></canvas>
            </div>
            <p>Distance: {dist.toFixed(0)} </p>
            <div class="progress-bar-wrapper">
                <div class="progress-bar" style:width={`${progress}%`} style:background-color="${progress_color}">
                    <br>
                </div>
            </div>
            <br>
            <button on:click={() => showModal = false} class ="modal-button">Close</button>
            <br>
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
    }
    .modal-map-wrapper img{
        width:100%;
        height:100%;
        position: relative;
    }
    .modal{
        border-radius: 12px;
        position: fixed;
        top:50%;
        left:50%;
        transform: translate(-50%, -50%);
        width: 600px;
        z-index: 999;
        background-color: white;
        overflow: hidden;
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
        object-fit: contain;
    }
    .map-wrapper {
        position: relative;
        border-radius: 12px;
        border: 1px solid black;
    }
    .map {
        transition: .5s;
        position: fixed;
        bottom: 0;
        right: 0;
        padding: 20px 18px;
    }
    .image-wrapper img {
        border-radius: 12px;
        width: 100%;
    }
    .image-wrapper {
        overflow: hidden;
        transition: 0.2s;
        transform-origin: bottom right;
        position: relative;
        opacity: .5;
        width:300px;
    }
    .image-wrapper:hover {
        cursor: crosshair;
        opacity: 1;
        width:400px;
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
    .button-guess {
        transition: 0.5s;
        padding: 5px;
        background-color: #ff9595;
        border: none;
        cursor: pointer;
        opacity: 0.7;
        border-radius: 12px;
        width: 100%;
    }
    .button-guess:hover {
        background-color: #990000;
        color: gold;
        opacity: 1;
    }
    .button-guess:disabled {
        background-color: transparent;
        width: 0%;
        height: 0%;
    }
    .progress-bar-wrapper {
        
        color: white;
        width: 100%;
        height: 100%;
        position: relative;
    }
    .progress-bar{
        transition: 1s;
        border-radius: 12px;
        height: 100%;
        background-color: red;
    }
    .modal-button{
        padding: 5px;
        background-color: #ff9595;
        border: none;
        cursor: pointer;
        opacity: 1;
        border-radius: 12px;
        width: 25%;
    }
    .modal-button:hover {
        background-color: #990000;
        color: gold;
        opacity: 1;
    }
</style>
