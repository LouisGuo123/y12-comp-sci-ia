<script lang="ts">
    import { onMount } from 'svelte';
    import * as THREE from 'three';
    import { Line2 } from 'three/examples/jsm/lines/Line2';
    import { LineGeometry } from 'three/examples/jsm/lines/LineGeometry';
    import { LineMaterial } from 'three/examples/jsm/lines/LineMaterial';
    import Katex from '$lib/components/Math/katex.svelte';

    let canvasElement: HTMLCanvasElement;
    let sliderElement: HTMLInputElement;
    let mathP: HTMLElement;
    let mathT: HTMLElement;
    let graphValue: HTMLElement;
    let graphValueMessage = '';

    onMount( () => {
        let [canvasWidth, canvasHeight] = [canvasElement.getBoundingClientRect().width, canvasElement.getBoundingClientRect().height];

        const scene = new THREE.Scene();
        const camera = new THREE.OrthographicCamera( -1, 1, 1 * canvasHeight / canvasWidth, -1 * canvasHeight / canvasWidth );
        const resolution = new THREE.Vector2(canvasWidth, canvasHeight);
        const renderer = new THREE.WebGLRenderer( {
            canvas: canvasElement,
            antialias: true,
            alpha: true,
        } );
        renderer.setClearColor( 0xffffff, 0 );

        const func: (x: number) => number = (x) => {
            return Math.sin(x);
        };
        const numOfPoints = 70;
        let points = new Array<number>( numOfPoints * 3 );
        const [lbound, rbound] = [-2 * Math.PI, 2 * Math.PI];
        const xScale = 1 / (2 * Math.PI);
        const yScale = .25;

        const gridHelper = new THREE.GridHelper( 30, 30, 0xffffff );
        gridHelper.rotateX(Math.PI / 2);
        gridHelper.scale.set(xScale, 1, yScale);
        
        scene.add( gridHelper );

        const functionLineGeometry = new LineGeometry();
        
        function updatePoints() {
            for (let i = 0; i < numOfPoints; i++) {
                let x = lbound + (rbound - lbound) / (numOfPoints - 1) * i;
                points[3 * i] = x;
                points[3 * i + 1] = func(x);
                points[3 * i + 2] = 0;
            }
            functionLineGeometry.setPositions( points );
        }
        updatePoints();

        const functionLineMaterial = new LineMaterial( {
            color: 0x26ad46,
            linewidth: 3,
            resolution: resolution,
        } );
        const functionLine = new Line2( functionLineGeometry, functionLineMaterial );
        functionLine.computeLineDistances();
        functionLine.scale.set(xScale, yScale, 1);
        
        scene.add( functionLine );

        let position = 0;
        const valueLineGeometry = new LineGeometry();
        valueLineGeometry.setPositions( [0, 0, 0, 0, 0, 0] );
        const deltaLineGeometry = new LineGeometry();
        deltaLineGeometry.setPositions( [0, 0, 0, 0, 0, 0] );
        const positionLineMaterial = new LineMaterial( {
            color: 0xffffff,
            linewidth: 2,
            resolution: resolution,
        } );
        const valueLine = new Line2( valueLineGeometry, positionLineMaterial );
        const deltaLine = new Line2( deltaLineGeometry, positionLineMaterial );
        let locked = false;

        const slopeLineGeometry = new LineGeometry();
        slopeLineGeometry.setPositions( [0, 0, 0, 0, 0, 0] );
        const slopeLineMaterial = new LineMaterial( {
            color: 0xffff00,
            linewidth: 2,
            resolution: resolution,
        } );
        const slopeLine = new Line2( slopeLineGeometry, slopeLineMaterial );

        valueLine.visible = false;
        deltaLine.visible = false;
        slopeLine.visible = false;

        scene.add(valueLine);
        scene.add(deltaLine);
        scene.add(slopeLine);

        camera.position.z = 1;

        function animate() {
            requestAnimationFrame( animate );
            renderer.render( scene, camera );
        }
        animate();

        function onCanvasClick(e: MouseEvent) {
            locked = !locked;
            if (!locked) onCanvasMove(e);
        }
        function onCanvasMove(e: MouseEvent): void {
            if (!locked) {
                let rect = canvasElement.getBoundingClientRect();
                let worldMousePos = new THREE.Vector3((e.x - rect.left) / canvasWidth * 2 - 1, (e.y - rect.top) / canvasHeight * 2 - 1, -1).unproject(camera);
                position = worldMousePos.x;
                valueLineGeometry.setPositions( [
                    position, camera.top, 0,
                    position, camera.bottom, 0,
                ] );
                updateDeltaLine();
                updateSlopeLine();
                updateText();
            }
        }
        function onCanvasEnter(): void {
            if (!locked) {
                valueLine.visible = true;
                deltaLine.visible = true;
                slopeLine.visible = true;
                graphValue.style.display = 'initial';
            }
        }
        function onCanvasLeave(): void {
            if (!locked) {
                valueLine.visible = false;
                deltaLine.visible = false;
                slopeLine.visible = false;
                graphValue.style.display = 'none';
            }
        }
        function updateText() {
            graphValueMessage = 't:' + (position / xScale).toFixed(2) + '\\\\p_1:' + func(position / xScale).toFixed(2) + '\\\\\\Delta:' + (+sliderElement.value).toFixed(2) + '\\\\p_2:' + func(position / xScale + +sliderElement.value).toFixed(2) + '\\\\\\frac{p_2 - p_1}{\\Delta}:' + ((func(position / xScale + +sliderElement.value) - func(position / xScale)) / +sliderElement.value).toFixed(2);
        }
        function updateDeltaLine(): void {
            deltaLineGeometry.setPositions( [
                position + +sliderElement.value * xScale, camera.top, 0,
                position + +sliderElement.value * xScale, camera.bottom, 0,
            ] );
        }
        function updateSlopeLine(): void {
            let slope = (func(position / xScale + +sliderElement.value) - func(position / xScale)) / +sliderElement.value;
            if (!isNaN(slope) && slope != null) {
                let lineFunc = (x: number) => slope * (x - position / xScale) + func(position / xScale);
                slopeLineGeometry.setPositions( [
                    camera.left, lineFunc(camera.left / xScale) * yScale, 0,
                    camera.right, lineFunc(camera.right / xScale) * yScale, 0,
                ] );
            }
            else {
                slopeLineGeometry.setPositions( [
                    0, 0, 0,
                    0, 0, 0,
                ] );
            }
        }
        function bindCanvasListeners(element: HTMLElement): void {
            element.onclick = onCanvasClick;
            element.onmousemove = onCanvasMove;
            element.onmouseenter = onCanvasEnter;
            element.onmouseleave = onCanvasLeave;
        }
        function updateResolution(): void {
            [canvasWidth, canvasHeight] = [canvasElement.getBoundingClientRect().width, canvasElement.getBoundingClientRect().height];
            resolution.x = canvasWidth;
            resolution.y = canvasHeight;
            functionLineMaterial.resolution = resolution;
            positionLineMaterial.resolution = resolution;
            slopeLineMaterial.resolution = resolution;
        }

        bindCanvasListeners(canvasElement);
        bindCanvasListeners(mathP);
        bindCanvasListeners(mathT);
        bindCanvasListeners(graphValue);
        sliderElement.oninput = () => {
            updateDeltaLine();
            updateSlopeLine();
            updateText();
        };
        window.addEventListener('resize', updateResolution);

        return () => {
            window.removeEventListener('resize', updateResolution);
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
    <br>
    <input bind:this={sliderElement} type="range" min="-2" max="2" step="0.01" value="1">
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

    input {
        width: 25%;
    }
</style>