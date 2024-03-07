<script lang="ts">
    import { onMount } from 'svelte';
    import * as THREE from 'three';
    import { LineGeometry } from 'three/examples/jsm/lines/LineGeometry';
    import { LineMaterial } from 'three/examples/jsm/lines/LineMaterial';
    import { Line2 } from 'three/examples/jsm/lines/Line2';
    import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

    const ORIGIN = new THREE.Vector3( 0, 0, 0 );
    const IHAT = new THREE.Vector3( 1, 0, 0 );
    const JHAT = new THREE.Vector3( 0, 1, 0 );
    const KHAT = new THREE.Vector3( 0, 0, 1 );

    let doneMounting = false;

    let canvasElement: HTMLCanvasElement;
    let [canvasWidth, canvasHeight] = [0, 0];

    let scene: THREE.Scene;
    let camera: THREE.PerspectiveCamera;
    let renderer: THREE.WebGLRenderer;
    let resolution: THREE.Vector2;
    let controls: OrbitControls;

    let gridHelper: THREE.GridHelper;
    let xArrow: THREE.ArrowHelper;
    let yArrow: THREE.ArrowHelper;
    let zArrow: THREE.ArrowHelper;
    let xLine: Line2;
    let yLine: Line2;
    let zLine: Line2;

    let vectorDataList = [
        {
            vector: new THREE.Vector3( 1, 1, -1 ),
            color: 0x00ffff,
        },
        {
            vector: new THREE.Vector3( 2, -1, 1 ),
            color: 0xffff00,
        },
        {
            vector: new THREE.Vector3( -1, 2, 3 ),
            color: 0xff00ff,
        },
        {
            vector: new THREE.Vector3( -4, 2, 0 ),
            color: 0xff8800,
        }
    ];
    let vectorArrowList = Array<THREE.ArrowHelper>();
    let vectorLineList = Array<Line2>();

    function animate(): void {
        requestAnimationFrame( animate );
        controls.target.clamp( new THREE.Vector3( -1, -1, -1 ), new THREE.Vector3( 1, 1, 1 ) );
        controls.update();

        renderer.render(scene, camera);
    }

    $: if ( doneMounting ) {
        resolution = new THREE.Vector2( canvasWidth, canvasHeight );

        xLine.material.resolution = resolution;
        yLine.material.resolution = resolution;
        zLine.material.resolution = resolution;

        for ( let vectorLine of vectorLineList ) {
            vectorLine.material.resolution = resolution;
        }
    }

    onMount( () => {
        doneMounting = true;

        function updateCanvasDimensions(): void {
            [canvasWidth, canvasHeight] = [canvasElement.getBoundingClientRect().width, canvasElement.getBoundingClientRect().height];
        }
        updateCanvasDimensions();

        scene = new THREE.Scene();
        camera = new THREE.PerspectiveCamera( 50, canvasWidth / canvasHeight );
        camera.position.set( 2.5, 2.5, 2.5 );
        camera.rotation.set( Math.PI / 4, Math.PI / 4, Math.PI / 4 );

        renderer = new THREE.WebGLRenderer( {
            canvas: canvasElement,
            antialias: true,
            alpha: true,
        } );
        renderer.setClearColor( 0xffffff, 0 );

        resolution = new THREE.Vector2( canvasWidth, canvasHeight );

        controls = new OrbitControls( camera, canvasElement );
        controls.enableDamping = true;
        controls.dampingFactor = .1;
        controls.minDistance = .5;
        controls.maxDistance = 13;

        gridHelper = new THREE.GridHelper( 10, 10, 0xffffff );

        xArrow = new THREE.ArrowHelper( IHAT, ORIGIN, 5 + .2, 0xff0000, .2, .15 );
        yArrow = new THREE.ArrowHelper( JHAT, ORIGIN, 5 + .2, 0x00ff00, .2, .15 );
        zArrow = new THREE.ArrowHelper( KHAT, ORIGIN, 5 + .2, 0x0000ff, .2, .15 );

        let xLineGeo = new LineGeometry();
        let yLineGeo = new LineGeometry();
        let zLineGeo = new LineGeometry();
        xLineGeo.setPositions( [
            0, 0, 0,
            5, 0, 0,
        ] );
        yLineGeo.setPositions( [
            0, 0, 0,
            0, 5, 0,
        ] );
        zLineGeo.setPositions( [
            0, 0, 0,
            0, 0, 5,
        ] );
        let xLineMat = new LineMaterial( {
            color: 0xff0000,
            linewidth: 1.5,
            resolution: resolution,
        } );
        let yLineMat = new LineMaterial( {
            color: 0x00ff00,
            linewidth: 1.5,
            resolution: resolution,
        } );
        let zLineMat = new LineMaterial( {
            color: 0x0000ff,
            linewidth: 1.5,
            resolution: resolution,
        } );
        xLine = new Line2( xLineGeo, xLineMat );
        yLine = new Line2( yLineGeo, yLineMat );
        zLine = new Line2( zLineGeo, zLineMat );
        
        scene.add( gridHelper );
        scene.add( xArrow );
        scene.add( yArrow );
        scene.add( zArrow );
        scene.add( xLine );
        scene.add( yLine );
        scene.add( zLine );

        for ( let vectorData of vectorDataList ) {
            let newVectorArrow = new THREE.ArrowHelper( vectorData.vector.clone().normalize(), new THREE.Vector3( 0, 0, 0 ), vectorData.vector.length(), vectorData.color, .2, .15 );
            let newVectorLineGeo = new LineGeometry();
            newVectorLineGeo.setPositions( [
                0, 0, 0,
                vectorData.vector.x - vectorData.vector.clone().normalize().x * .2, vectorData.vector.y - vectorData.vector.clone().normalize().y * .2, vectorData.vector.z - vectorData.vector.clone().normalize().z * .2,
            ] );
            let newVectorLineMat = new LineMaterial( {
                color: vectorData.color,
                linewidth: 1.5,
                resolution: resolution,
            } );
            let newVectorLine = new Line2( newVectorLineGeo, newVectorLineMat );

            vectorArrowList.push( newVectorArrow );
            vectorLineList.push( newVectorLine );

            scene.add( newVectorArrow );
            scene.add( newVectorLine );
        }

        animate();

        window.addEventListener( 'resize', updateCanvasDimensions );
        
        return () => {
            window.removeEventListener( 'resize', updateCanvasDimensions );
        };
    } );

    function checkAnswer(index: number) {
        let xInput = document.getElementById( 'vectorInput.x.' + index )!;
        let yInput = document.getElementById( 'vectorInput.y.' + index )!;
        let zInput = document.getElementById( 'vectorInput.z.' + index )!;
        if ( xInput instanceof HTMLInputElement && vectorDataList[index].vector.x == +xInput.value
            && yInput instanceof HTMLInputElement && vectorDataList[index].vector.y == +yInput.value
            && zInput instanceof HTMLInputElement && vectorDataList[index].vector.z == +zInput.value ) {
            xInput.style.background = '#00ff00';
            yInput.style.background = '#00ff00';
            zInput.style.background = '#00ff00';
            xInput.style.color = '#000000';
            yInput.style.color = '#000000';
            zInput.style.color = '#000000';
        } else {
            xInput.style.background = '#ff0000';
            yInput.style.background = '#ff0000';
            zInput.style.background = '#ff0000';
            xInput.style.color = '#ffffff';
            yInput.style.color = '#ffffff';
            zInput.style.color = '#ffffff';
        }
    }
</script>

<div class="wrapper">
    <canvas bind:this={canvasElement} width="1280" height="720"></canvas>
    <div class="check-wrapper">
        {#each vectorDataList as vectorData, i}
            <div class="input-wrapper">
                <span style={'background: #' + vectorData.color.toString( 16 ).padStart( 6, '0' )}></span>
                <input type="number" min="-5" max="5" step="1" id={'vectorInput.x.' + i}>
                <input type="number" min="-5" max="5" step="1" id={'vectorInput.y.' + i}>
                <input type="number" min="-5" max="5" step="1" id={'vectorInput.z.' + i}>
                <button on:click={() => checkAnswer( i )}>Check Answer</button>
            </div>
        {/each}
    </div>
</div>

<style>
    .wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;

        font: normal 100 calc(1.8vh + 3px) 'Roboto', sans-serif;
    }

    canvas {
        margin: 1.4em 0;

        width: calc(25vw + 180px);
        height: auto;
    }

    .check-wrapper {
        display: flex;
        flex-direction: column;
        gap: calc(.5vh + 5px);
        margin-bottom: 1.4em;
    }

    .input-wrapper {
        display: flex;
        gap: calc(1vw + 10px);
    }

    input[type=number] {
        width: calc(30px + 1vw);
        height: 2em;
        background: #ffffff;
        text-align: center;
    }

    span {
        display: block;
        width: calc(20px + 2vw);
        height: 2em;
    }

    button {
        width: calc(110px + 3vw);
        height: 2em;
        background: #e0e0e0;
        border-radius: calc(.1vw + 5px);
        text-align: center;
    }
</style>