<script lang="ts">
    import { onMount } from 'svelte';
    import * as THREE from 'three';
    import { DragControls } from 'three/examples/jsm/controls/DragControls';
    import { LineGeometry } from 'three/examples/jsm/lines/LineGeometry';
    import { LineMaterial } from 'three/examples/jsm/lines/LineMaterial';
    import { Line2 } from 'three/examples/jsm/lines/Line2';

    function isOnScreen( element: HTMLElement ): boolean {
        let elementRect = element.getBoundingClientRect();

        return elementRect.bottom > 0 && elementRect.top < window.innerHeight;
    }

    const ORIGIN = new THREE.Vector3( 0, 0, 0 );
    const LOCAL_X_FACTOR = .2;
    const LOCAL_Y_FACTOR = .2;

    function worldToLocal( vector: THREE.Vector3 ) {
        return vector.clone().multiply( new THREE.Vector3( 1 / LOCAL_X_FACTOR, 1 / LOCAL_Y_FACTOR, 1 ) );
    }

    function localToWorld( vector: THREE.Vector3 ) {
        return vector.clone().multiply( new THREE.Vector3( LOCAL_X_FACTOR, LOCAL_Y_FACTOR, 1 ) );
    }

    let doneMounting = false;

    let canvasElement: HTMLCanvasElement;
    let [canvasWidth, canvasHeight] = [0, 0];
    let resolution: THREE.Vector2;

    let scene: THREE.Scene;
    let camera: THREE.Camera;
    let renderer: THREE.WebGLRenderer;
    
    let controls: DragControls;

    let gridHelper: THREE.GridHelper;
    let transformedGridHelper: THREE.GridHelper;

    let draggableObjects = new Array<THREE.Object3D>(0);
    let ballToArrowMap = new Map<THREE.Object3D, [THREE.Vector3, Line2, THREE.ArrowHelper, number, number]>();

    let ball1: THREE.Mesh;
    let ball2: THREE.Mesh;
    let ball3: THREE.Mesh;

    let transformedLine: Line2;
    let transformedArrowHelper: THREE.ArrowHelper;

    function getFatArrow( src: THREE.Vector3, dest: THREE.Vector3, arrowColor: THREE.ColorRepresentation, lineWidth: number, headLength: number, headWidth: number ): [Line2, THREE.ArrowHelper] {
        let arrowHelper = new THREE.ArrowHelper( dest.clone().sub( src ).normalize(), src, dest.clone().sub( src ).length(), arrowColor, headLength, headWidth );
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
        (arrowHelper.cone.material as THREE.Material).depthTest = false;
        (arrowHelper.line.material as THREE.Material).depthTest = false;

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

    function onObjectDrag( event: { object: THREE.Object3D<THREE.Object3DEventMap> } & THREE.Event<'drag', DragControls> ) {
        let [src, line, arrowHelper, headWidth, headLength] = ballToArrowMap.get( event.object )!;
        
        updateFatArrow( src, event.object.position, line, arrowHelper, headLength, headWidth );
        updateTransformedGridHelper();
        updateTransformedArrow();
    }

    function updateTransformedGridHelper() {
        transformedGridHelper.matrix.set(
            ball2.position.x, 0, ball3.position.x, 0,
            ball2.position.y, 0, ball3.position.y, 0,
            ball2.position.z, 1, ball3.position.z, 0,
            0, 0, 0, 1,
        );
    };

    function updateTransformedArrow() {
        updateFatArrow( ORIGIN, ball2.position.clone().multiplyScalar( ball1.position.x / LOCAL_X_FACTOR ).add(ball3.position.clone().multiplyScalar( ball1.position.y / LOCAL_Y_FACTOR )), transformedLine, transformedArrowHelper, .05, .05 );
    }

    function updateCanvasDimensions(): void {
        [canvasWidth, canvasHeight] = [canvasElement.getBoundingClientRect().width, canvasElement.getBoundingClientRect().height];
        resolution = new THREE.Vector2( canvasWidth, canvasHeight );
    }

    function animate(): void {
        requestAnimationFrame( animate );

        if ( isOnScreen( canvasElement ) ) {
            renderer.render( scene, camera );
        }
    }

    $: if ( doneMounting ) {
        for ( let [ball, [dest, line]] of ballToArrowMap ) {
            line.material.resolution = resolution;
        }
        transformedLine.material.resolution = resolution;
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

        let line: Line2;
        let arrowHelper: THREE.ArrowHelper;
        [line, arrowHelper, ball1] = getDraggableFatArrow( ORIGIN, localToWorld( new THREE.Vector3( 2, 1, 0 ) ), 0xffff00, 3, .05, .05 );
        updateFatArrowRenderOrder( line, arrowHelper, 1 );

        let matrixLine1: Line2;
        let matrixArrowHelper1: THREE.ArrowHelper;
        [matrixLine1, matrixArrowHelper1, ball2] = getDraggableFatArrow( ORIGIN, localToWorld( new THREE.Vector3( 1, 0, 0 ) ), 0xff0000, 3, .05, .05 );
        updateFatArrowRenderOrder( matrixLine1, matrixArrowHelper1, 2 );

        let matrixLine2: Line2;
        let matrixArrowHelper2: THREE.ArrowHelper;
        [matrixLine2, matrixArrowHelper2, ball3] = getDraggableFatArrow( ORIGIN, localToWorld( new THREE.Vector3( -1, 1, 0 ) ), 0x00ff00, 3, .05, .05 );
        updateFatArrowRenderOrder( matrixLine2, matrixArrowHelper2, 2 );

        draggableObjects.push( ball1 );
        draggableObjects.push( ball2 );
        draggableObjects.push( ball3 );

        scene.add( line, arrowHelper, ball1 );
        scene.add( matrixLine1, matrixArrowHelper1, ball2 );
        scene.add( matrixLine2, matrixArrowHelper2, ball3 );

        [transformedLine, transformedArrowHelper] = getFatArrow( ORIGIN, ball1.position, 0xff00ff, 3, .05, .05 );
        updateFatArrowRenderOrder( transformedLine, transformedArrowHelper, 1 );

        updateTransformedArrow();

        scene.add( transformedLine, transformedArrowHelper );

        gridHelper = new THREE.GridHelper( 10, 10, 0xffffff );
        gridHelper.rotateX( Math.PI / 2 );
        gridHelper.scale.set( LOCAL_X_FACTOR, 1, LOCAL_Y_FACTOR );
        
        scene.add( gridHelper );

        transformedGridHelper = new THREE.GridHelper( 20, 20, 0x40c0ff, 0x506088 );
        transformedGridHelper.matrixAutoUpdate = false;
        updateTransformedGridHelper();

        scene.add( transformedGridHelper );

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
</style>