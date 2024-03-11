<script lang="ts">
    import { onMount } from 'svelte';
    import * as THREE from 'three';
    import { DragControls } from 'three/examples/jsm/controls/DragControls';

    function isOnScreen( element: HTMLElement ): boolean {
        let elementRect = element.getBoundingClientRect();

        return elementRect.bottom > 0 && elementRect.top < window.innerHeight;
    }

    const ORIGIN = new THREE.Vector3( 0, 0, 0 );
    const IHAT = new THREE.Vector3( 1, 0, 0 );
    const JHAT = new THREE.Vector3( 0, 1, 0 );

    /* https://stackoverflow.com/questions/25582882/javascript-math-random-normal-distribution-gaussian-bell-curve */
    function gaussian( mu: number, sigma: number ) {
        const u = 1 - Math.random();
        const v = Math.random();
        const z = Math.sqrt( -2.0 * Math.log( u ) ) * Math.cos( 2.0 * Math.PI * v );

        return z * sigma + mu;
    }

    function randomRange( min: number, max: number ) {
        return Math.random() * (max - min) + min;
    }

    function clamp( value: number, min: number, max: number ) {
        return Math.min( Math.max( value, min ), max );
    }

    let doneMounting = false;

    let canvasElement: HTMLCanvasElement;
    let [canvasWidth, canvasHeight] = [0, 0];

    let scene: THREE.Scene;
    let camera: THREE.Camera;
    let renderer: THREE.WebGLRenderer;
    let resolution: THREE.Vector2;
    
    let controls: DragControls;

    let gridHelper: THREE.GridHelper;

    let draggableObjects = new Array<THREE.Object3D>( 0 );
    let dotToBucketMap = new Map<THREE.Object3D, {x: number, y: number}>();
    let buckets = new Array<Array<{count: number, plane: THREE.Mesh}>>( 10 );
    let [startX, startY] = [-1, -.6];
    let [bucketWidth, bucketHeight] = [.2, .2];

    for (let x = 0; x < buckets.length; x++) {
        buckets[x] = new Array<{count: number, plane: THREE.Mesh}>( 6 );
    }

    function onObjectDrag( event: { object: THREE.Object3D<THREE.Object3DEventMap> } & THREE.Event<'drag', DragControls> ): void {
        updateDot( event.object, event.object.position.x, event.object.position.y );
    }

    function createDot( x: number, y: number, radius: number ): THREE.Mesh {
        let dotGeo = new THREE.SphereGeometry( radius );
        let dotMat = new THREE.MeshBasicMaterial( {
            color: 0xffffff
        } );
        let dot = new THREE.Mesh( dotGeo, dotMat );
        dot.position.set( x, y, 0 );
        updateDot( dot, x, y );
        draggableObjects.push( dot );

        return dot;
    }

    function updateDot( dot: THREE.Object3D, x: number, y: number ): void {
        if ( startX <= x && x <= startX + bucketWidth * buckets.length && startY <= y && y <= startY + bucketHeight * buckets[0].length ) {
            if (!dotToBucketMap.has( dot )) {
                dotToBucketMap.set( dot, {x: 0, y: 0} );
                buckets[0][0].count++;
            }

            let oldBucketX = dotToBucketMap.get( dot )!.x;
            let oldBucketY = dotToBucketMap.get( dot )!.y;

            let bucketX = Math.floor((x - startX) / bucketWidth);
            let bucketY =  Math.floor((y - startY) / bucketHeight);

            if ( bucketX != oldBucketX || bucketY != oldBucketY ) {
                buckets[oldBucketX][oldBucketY].count--;
                buckets[bucketX][bucketY].count++;

                updateBucketPlane( oldBucketX, oldBucketY );
                updateBucketPlane( bucketX, bucketY );

                dotToBucketMap.set( dot, {x: bucketX, y: bucketY} );
            }
        }
    }

    function updateBucketPlane( bucketX: number, bucketY: number ) {
        let color = clamp( buckets[bucketX][bucketY].count * 16, 0, 255 );

        (buckets[bucketX][bucketY].plane.material as THREE.MeshBasicMaterial).color = new THREE.Color( (color << 16) | (color << 8) | color );
    }

    function animate(): void {
        requestAnimationFrame( animate );

        if ( isOnScreen( canvasElement ) ) {
            renderer.render(scene, camera);
        }
    }

    onMount( () => {
        function updateCanvasDimensions(): void {
            [canvasWidth, canvasHeight] = [canvasElement.getBoundingClientRect().width, canvasElement.getBoundingClientRect().height];
            resolution = new THREE.Vector2( canvasWidth, canvasHeight );
        }
        updateCanvasDimensions();

        scene = new THREE.Scene();
        camera = new THREE.OrthographicCamera( -1, 1, canvasHeight / canvasWidth, -canvasHeight / canvasWidth );
        camera.position.z = 1;

        renderer = new THREE.WebGLRenderer( {
            canvas: canvasElement,
            antialias: true,
            alpha: true,
        } );
        renderer.sortObjects = true;
        renderer.setClearColor( 0xffffff, 0 );

        controls = new DragControls( draggableObjects, camera, canvasElement );
        controls.addEventListener( 'drag', ( event ) => onObjectDrag( event ) );

        for (let x = 0; x < buckets.length; x++) {
            for (let y = 0; y < buckets[0].length; y++) {
                let planeGeo = new THREE.PlaneGeometry( .2, .2 );
                let planeMat = new THREE.MeshBasicMaterial( {
                    color: 0x000000
                } );
                let plane = new THREE.Mesh( planeGeo, planeMat );
                plane.position.set( x * bucketWidth + startX + .1, y * bucketHeight + startY + .1, 0);

                buckets[x][y] = { count: 0, plane: plane };
                scene.add( plane );
            }
        }

        gridHelper = new THREE.GridHelper( 10, 10, 0xffffff );
        gridHelper.rotateX( Math.PI / 2 );
        gridHelper.scale.set( .2, 1, .2 );
        
        scene.add( gridHelper );

        for (let i = 0; i < 40; i++) {
            let x = randomRange( -.9, .9 );
            let y = clamp( x / 2 + gaussian( 0, .1 ), -.5, .5 );

            scene.add( createDot( x, y, .02 ) );
        }

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
    <caption>2D Histogram (Drag the Points!)</caption>
</div>

<style>
    .wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;

        color: white;
    }

    canvas {
        margin: 1.4em 0;

        width: calc(25vw + 180px);
        height: auto;
    }
</style>