<script lang="ts">
    import { onMount } from 'svelte';
    import * as THREE from 'three';
    import { DragControls } from 'three/examples/jsm/controls/DragControls';
    import { LineGeometry } from 'three/examples/jsm/lines/LineGeometry';
    import { LineMaterial } from 'three/examples/jsm/lines/LineMaterial';
    import { Line2 } from 'three/examples/jsm/lines/Line2';
    import Katex from '$lib/components/Math/katex.svelte';

    function isOnScreen( element: HTMLElement ): boolean {
        let elementRect = element.getBoundingClientRect();

        return elementRect.bottom > 0 && elementRect.top < window.innerHeight;
    }

    const ORIGIN = new THREE.Vector3( 0, 0, 0 );
    const IHAT = new THREE.Vector3( 1, 0, 0 );
    const JHAT = new THREE.Vector3( 0, 1, 0 );

    let doneMounting = false;

    let canvasElement: HTMLCanvasElement;
    let [canvasWidth, canvasHeight] = [0, 0];

    let scene: THREE.Scene;
    let camera: THREE.Camera;
    let renderer: THREE.WebGLRenderer;
    let resolution: THREE.Vector2;
    
    let controls: DragControls;

    let gridHelper: THREE.GridHelper;

    let draggableObjects = new Array<THREE.Object3D>(0);
    let ballToArrowMap = new Map<THREE.Object3D, [THREE.Vector3, Line2, THREE.ArrowHelper, number, number]>();

    let ball1: THREE.Mesh;

    let scaledLine: Line2;
    let scaledArrowHelper: THREE.ArrowHelper;

    let scalar: number = 2;

    function getFatArrow( src: THREE.Vector3, dest: THREE.Vector3, arrowColor: THREE.ColorRepresentation, lineWidth: number, headLength: number, headWidth: number ): [Line2, THREE.ArrowHelper] {
        let arrowHelper = new THREE.ArrowHelper( dest.clone().sub( src ).normalize(), src, dest.clone().sub( src ).length(), arrowColor, headLength, headWidth );
        (arrowHelper.cone.material as THREE.Material).depthTest = false;
        (arrowHelper.line.material as THREE.Material).depthTest = false;

        let lineGeo = new LineGeometry();
        let headOffset = dest.clone().sub(src).normalize().multiplyScalar( headLength );
        lineGeo.setPositions( [
            src.x, src.y, src.z,
            dest.x - headOffset.x, dest.y - headOffset.y, dest.z - headOffset.z,
        ] );
        let lineMat = new LineMaterial( {
            color: arrowColor,
            linewidth: lineWidth,
            resolution: resolution,
            depthTest: false,
        } );

        let line = new Line2( lineGeo, lineMat );

        return [line, arrowHelper];
    }

    function getDraggableFatArrow( src: THREE.Vector3, dest: THREE.Vector3, arrowColor: THREE.ColorRepresentation, lineWidth: number, headLength: number, headWidth: number ): [Line2, THREE.ArrowHelper, THREE.Mesh] {
        let [line, arrowHelper] = getFatArrow( src, dest, arrowColor, lineWidth, headLength, headWidth );

        let ballGeo = new THREE.SphereGeometry( .1 );
        let ballMat = new THREE.MeshBasicMaterial( {
            opacity: 0,
            transparent: true,
        } );
        let ball = new THREE.Mesh( ballGeo, ballMat );
        ball.position.set( dest.x, dest.y, dest.z );
        ballToArrowMap.set( ball, [src, line, arrowHelper, headLength, headWidth] );

        return [line, arrowHelper, ball];
    }

    function updateFatArrow( src: THREE.Vector3, dest: THREE.Vector3, line: Line2, arrowHelper: THREE.ArrowHelper, headLength: number, headWidth: number ): void {
        let headOffset = dest.clone().sub(src).normalize().multiplyScalar( headLength );
        line.geometry.setPositions( [
            src.x, src.y, src.z,
            dest.x - headOffset.x, dest.y - headOffset.y, dest.z - headOffset.z,
        ] );

        arrowHelper.position.set( src.x, src.y, src.z );
        arrowHelper.setDirection( dest.clone().sub( src ).normalize() );
        arrowHelper.setLength( dest.clone().sub( src ).length(), headLength, headWidth );
    }

    function updateFatArrowRenderOrder( line: Line2, arrowHelper: THREE.ArrowHelper, renderOrder: number ): void {
        line.renderOrder = renderOrder;
        arrowHelper.cone.renderOrder = renderOrder;
        arrowHelper.line.renderOrder = renderOrder;
    }

    function updateCanvasDimensions(): void {
        [canvasWidth, canvasHeight] = [canvasElement.getBoundingClientRect().width, canvasElement.getBoundingClientRect().height];
        resolution = new THREE.Vector2( canvasWidth, canvasHeight );
    }

    function onObjectDrag( event: { object: THREE.Object3D<THREE.Object3DEventMap> } & THREE.Event<'drag', DragControls> ) {
        let [src, line, arrowHelper, headWidth, headLength] = ballToArrowMap.get( event.object )!;
        
        updateFatArrow( src, event.object.position, line, arrowHelper, headLength, headWidth );
        updateFatArrow( ORIGIN, ball1.position.clone().multiplyScalar( scalar ), scaledLine, scaledArrowHelper, headWidth, headLength );
    }

    function animate(): void {
        requestAnimationFrame( animate );

        if ( isOnScreen( canvasElement ) ) {
            renderer.render(scene, camera);
        }
    }

    $: if ( doneMounting ) {
        updateFatArrow( ORIGIN, ball1.position.clone().multiplyScalar( scalar ), scaledLine, scaledArrowHelper, .05, .05 );
        if ( scalar <= 1 ) {
            updateFatArrowRenderOrder( scaledLine, scaledArrowHelper, 1 );
        } else {
            updateFatArrowRenderOrder( scaledLine, scaledArrowHelper, -1 );
        }
    }

    $: if ( doneMounting ) {
        for ( let [ball, [src, line]] of ballToArrowMap ) {
            line.material.resolution = resolution;
        }
        scaledLine.material.resolution = resolution;
    }

    onMount( () => {
        updateCanvasDimensions();

        scene = new THREE.Scene();
        camera = new THREE.OrthographicCamera( -1, 1, canvasHeight / canvasWidth, -canvasHeight / canvasWidth );
        camera.position.z = 1;

        renderer = new THREE.WebGLRenderer( {
            canvas: canvasElement,
            antialias: true,
            alpha: true,
        } );
        renderer.setClearColor( 0xffffff, 0 );

        controls = new DragControls( draggableObjects, camera, canvasElement );
        controls.addEventListener( 'drag', ( event ) => onObjectDrag( event ) );

        gridHelper = new THREE.GridHelper( 10, 10, 0xffffff );
        gridHelper.rotateX( Math.PI / 2 );
        gridHelper.scale.set( .2, 1, .2 );
        
        scene.add(gridHelper);

        let line1: Line2;
        let arrowHelper1: THREE.ArrowHelper;
        [line1, arrowHelper1, ball1] = getDraggableFatArrow( ORIGIN, new THREE.Vector3( .4, .2, 0 ), 0xffff00, 3, .05, .05 );

        draggableObjects.push( ball1 );

        [scaledLine, scaledArrowHelper] = getFatArrow( ORIGIN, ball1.position.clone().multiplyScalar( scalar ), 0xff00ff, 3, .05, .05 );

        scene.add( line1, arrowHelper1, ball1 );
        scene.add( scaledLine, scaledArrowHelper );

        animate();

        window.addEventListener( 'resize', updateCanvasDimensions );
        
        doneMounting = true;

        return () => {
            window.removeEventListener( 'resize', updateCanvasDimensions );
        };
    } );
</script>

<div class="wrapper">
    <canvas bind:this={canvasElement} width="1280" height="720"></canvas>
    <input type="range" min="-2" max="2" step=".01" value="2" on:input={(e) => {scalar = +e.currentTarget.value}}>
    <p class="article-p"><Katex math={'s = ' + scalar.toString()} /></p>
</div>

<style>
    .wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;

        margin-bottom: 1.4em;
        font: normal 100 calc(1.8vh + 3px) 'Roboto', sans-serif;
    }

    canvas {
        margin: 1.4em 0;

        width: calc(25vw + 180px);
        height: auto;
    }

    input[type="range"] {
        width: calc(20vw + 10vh);
    }

    .article-p.article-p {
        text-indent: 0;
    }
</style>