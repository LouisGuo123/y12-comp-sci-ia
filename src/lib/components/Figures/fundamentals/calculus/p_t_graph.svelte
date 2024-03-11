<script lang="ts">
    import { onMount } from 'svelte';
    import * as THREE from 'three';
    import { Line2 } from 'three/examples/jsm/lines/Line2';
    import { LineGeometry } from 'three/examples/jsm/lines/LineGeometry';
    import { LineMaterial } from 'three/examples/jsm/lines/LineMaterial';
    import Katex from '$lib/components/Math/katex.svelte';

    function isOnScreen( element: HTMLElement ): boolean {
        let elementRect = element.getBoundingClientRect();

        return elementRect.bottom > 0 && elementRect.top < window.innerHeight;
    }

    const numOfPoints = 70;
    const [lbound, rbound] = [-2 * Math.PI, 2 * Math.PI];
    const xScale = 1 / (2 * Math.PI);
    const yScale = .25;

    let func: (x: number) => number = (x) => {
        return Math.sin(x);
    };
    let doneMounting = false;

    let canvasElement: HTMLCanvasElement;
    let [canvasWidth, canvasHeight] = [0, 0];

    let mathP: HTMLElement;
    let mathT: HTMLElement;
    let graphValue: HTMLElement;
    let graphValueMessage = '';

    let scene: THREE.Scene;
    let camera: THREE.OrthographicCamera;
    let renderer: THREE.WebGLRenderer;
    let resolution: THREE.Vector2;

    let gridHelper: THREE.GridHelper;

    let functionLine: Line2;

    let valueLine: Line2;

    function onCanvasMove(e: MouseEvent): void {
        let rect = canvasElement.getBoundingClientRect();
        let worldMousePos = new THREE.Vector3((e.x - rect.left) / canvasWidth * 2 - 1, (e.y - rect.top) / canvasHeight * 2 - 1, -1).unproject(camera);
        valueLine.geometry.setPositions( [
            worldMousePos.x, camera.top, 0,
            worldMousePos.x, camera.bottom, 0,
        ] );
        let graphMousePos = worldMousePos.multiply(new THREE.Vector3(1 / xScale, 1 / yScale, 1));
        graphValueMessage = 't:' + graphMousePos.x.toFixed(2) + '\\\\p:' + func(graphMousePos.x).toFixed(2);
    }

    function onCanvasEnter(): void {
        valueLine.visible = true;
        graphValue.style.display = 'initial';
    }

    function onCanvasLeave(): void {
        valueLine.visible = false;
        graphValue.style.display = 'none';
    }

    function bindCanvasListeners(element: HTMLElement): void {
        element.onmousemove = onCanvasMove;
        element.onmouseenter = onCanvasEnter;
        element.onmouseleave = onCanvasLeave;
    }

    function updateCanvasDimensions(): void {
        [canvasWidth, canvasHeight] = [canvasElement.getBoundingClientRect().width, canvasElement.getBoundingClientRect().height];
        resolution = new THREE.Vector2( canvasWidth, canvasHeight );
    }
    
    function animate() {
        requestAnimationFrame( animate );

        if ( isOnScreen( canvasElement ) ) {
            renderer.render( scene, camera );
        }
    }

    $: if ( doneMounting ) {
        functionLine.material.resolution = resolution;
        valueLine.material.resolution = resolution;
    }

    onMount( () => {
        updateCanvasDimensions();

        scene = new THREE.Scene();
        camera = new THREE.OrthographicCamera( -1, 1, 1 * canvasHeight / canvasWidth, -1 * canvasHeight / canvasWidth );
        renderer = new THREE.WebGLRenderer( {
            canvas: canvasElement,
            antialias: true,
            alpha: true,
        } );
        renderer.setClearColor( 0xffffff, 0 );

        gridHelper = new THREE.GridHelper( 30, 30, 0xffffff );
        gridHelper.rotateX(Math.PI / 2);
        gridHelper.scale.set(xScale, 1, yScale);
        
        scene.add(gridHelper);

        let functionLineGeometry = new LineGeometry();
        let points = new Array<number>(numOfPoints * 3);
        
        for (let i = 0; i < numOfPoints; i++) {
            let x = lbound + (rbound - lbound) / (numOfPoints - 1) * i;
            points[3 * i] = x;
            points[3 * i + 1] = func(x);
            points[3 * i + 2] = 0;
        }
        functionLineGeometry.setPositions(points);

        let functionLineMaterial = new LineMaterial( {
            color: 0x26ad46,
            linewidth: 3,
            resolution: resolution,
        } );
        functionLine = new Line2(functionLineGeometry, functionLineMaterial);
        functionLine.computeLineDistances();
        functionLine.scale.set(xScale, yScale, 1);
        
        scene.add(functionLine);

        let valueLineGeometry = new LineGeometry();
        valueLineGeometry.setPositions( [0, 0, 0, 0, 0, 0] );
        let valueLineMaterial = new LineMaterial( {
            color: 0xffffff,
            linewidth: 2,
            resolution: resolution,
        } );
        valueLine = new Line2(valueLineGeometry, valueLineMaterial);

        valueLine.visible = false;

        scene.add(valueLine);

        camera.position.z = 1;

        bindCanvasListeners(canvasElement);
        bindCanvasListeners(mathP);
        bindCanvasListeners(mathT);
        bindCanvasListeners(graphValue);
        window.addEventListener('resize', updateCanvasDimensions);

        animate();

        doneMounting = true;
        
        return () => {
            window.removeEventListener('resize', updateCanvasDimensions);
        };
    } );
</script>

<div class="wrapper">
    <div>
        <p bind:this={mathP} class="math-p article-p"><Katex math={'p'} /></p>
        <p bind:this={mathT} class="math-t article-p"><Katex math={'t'} /></p>
        <p bind:this={graphValue} class="graph-value article-p" style="display: none;"><Katex bind:math={graphValueMessage} /></p>
        <canvas bind:this={canvasElement} width="1280" height="720"></canvas>
    </div>
</div>

<style>
    .wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .article-p.article-p {
        position: absolute;

        text-indent: 0;
        user-select: none;
    }

    .math-p {
        transform: translate(calc((25vw + 180px) * .45), 0);
    }

    .math-t {
        transform: translate(calc((25vw + 180px) * .94), calc((25vw + 180px) * .28));
    }

    .graph-value {
        transform: translate(10px, 10px);
    }

    canvas {
        width: calc(25vw + 180px);
        height: auto;
    }
</style>