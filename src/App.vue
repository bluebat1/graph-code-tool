<script setup>
</script>

<template>
	<div style="background: #000;margin: 0px;padding: 0px;">
		<div class="graphContainer" ref="graphContainer" width="600">
		</div>
		<div class="toolBar" ref="toolBar"></div>


		<div class="btn" @click="test">
			test
		</div>
	</div>
</template>

<script>
// import { mxGraph } from 'mxgraph-js'

import { Graph, InternalEvent, ModelXmlSerializer, MaxToolbar, Cell, Geometry, registerAllCodecs,cellArrayUtils,gestureUtils } from '@maxgraph/core';

export default {
	data() {
		return {
			graph: null,
		}
	},
	mounted() {


		const container = this.$refs.graphContainer;
		// 禁用默认菜单事件
		InternalEvent.disableContextMenu(container);

		// 创建画布
		const graph = new Graph(container);
		graph.setPanning(true); // Use mouse right button for panning
		registerAllCodecs(true)

		// 画布根节点
		const parent = graph.getDefaultParent();


		// 工具栏
		const toolbarAndGraphParentContainer = this.$refs.toolBar;
		toolbarAndGraphParentContainer.style.display = 'flex';


		// Creates the div for the toolbar
		const tbContainer = document.createElement('div');
		tbContainer.style.display = 'flex';
		tbContainer.style.flexDirection = 'column';
		tbContainer.style.marginRight = '.5rem';
		toolbarAndGraphParentContainer.appendChild(tbContainer);

		// Creates new toolbar without event processing
		const toolbar = new MaxToolbar(tbContainer);
		toolbar.enabled = false;

		// Enables new connections in the graph
		graph.setConnectable(true);
		graph.setMultigraph(false);

		// Adds cells to the model in a single step
		graph.batchUpdate(() => {
			const vertex01 = graph.insertVertex({
				parent,
				position: [10, 10],
				size: [100, 100],
				value: 'rectangle',
			});
			const vertex02 = graph.insertVertex({
				parent,
				position: [350, 90],
				size: [50, 50],
				style: {
					fillColor: 'orange',
					shape: 'ellipse',
					verticalAlign: 'top',
					verticalLabelPosition: 'bottom',
				},
				value: 'ellipse',
			});
			graph.insertEdge({
				parent,
				source: vertex01,
				target: vertex02,
				value: 'edge',
				style: {
					edgeStyle: 'orthogonalEdgeStyle',
					rounded: true,
				},
			});
		});
		this.graph = graph;

		function addCell(isVertex, icon, w, h, style) {
			const cell = new Cell(null, new Geometry(0, 0, w, h), style);
			cell.setVertex(isVertex);
			cell.setEdge(!isVertex);
			addToolbarItem(graph, toolbar, cell, icon, !isVertex ? 'Edge' : undefined);
		}
		function addVertex(icon, w, h, style) {
			addCell(true, icon, w, h, style);
		}
 
		// the line is not correctly display when calling toolbar.addLine() when the toolbar is in a flex container
		function addToolbarLine() {
			const hr = document.createElement('hr');
			hr.style.maxHeight = '0';
			hr.style.minWidth = '100%';
			hr.setAttribute('size', '1');
			tbContainer.appendChild(hr);
		}
		addVertex('/images/swimlane.gif', 120, 160, {
			shape: 'swimlane',
			startSize: 20
		});
		addVertex('/images/rectangle.gif', 100, 40, {});
		addVertex('/images/rounded.gif', 100, 40, {
			rounded: true
		});
		addVertex('/images/ellipse.gif', 40, 40, {
			shape: 'ellipse'
		});
		addVertex('/images/rhombus.gif', 40, 40, {
			shape: 'rhombus'
		});
		addVertex('/images/triangle.gif', 40, 40, {
			shape: 'triangle'
		});
		addVertex('/images/cylinder.gif', 40, 40, {
			shape: 'cylinder'
		});
		addVertex('/images/actor.gif', 30, 40, {
			shape: 'actor'
		});
		addToolbarLine();
		function addEdge(icon, w, h, style) {
			addCell(false, icon, w, h, style);
		}
		addEdge('/images/entity.gif', 50, 50, {});
		addToolbarLine();

		function addToolbarItem(graph , toolbar , prototype , image ) {
			// Function that is executed when the image is dropped on the graph.
			// The cell argument points to the cell under the mouse pointer if there is one.
			const dropHandler = (graph , _evt , _cell , x , y ) => {
				graph.stopEditing(false);
				const vertex = cellArrayUtils.cloneCell(prototype);
				if (vertex?.geometry) {
					x !== undefined && (vertex.geometry.x = x);
					y !== undefined && (vertex.geometry.y = y);
				}
				graph.addCell(vertex, null);
				graph.setSelectionCell(vertex);
			};

			// Creates the image which is used as the drag icon (preview)
			const img = toolbar.addMode(null, image, (evt , cell ) => {
				const pt = graph.getPointForEvent(evt);
				dropHandler(graph, evt, cell, pt.x, pt.y);
			}, '');
			InternalEvent.addListener(img, 'mousedown', (evt ) => {
				if (img.enabled == false) {
					InternalEvent.consume(evt);
				}
			});
			gestureUtils.makeDraggable(img, graph, dropHandler);
			return img;
		}


		// 缩放事件
		this.$refs.graphContainer.addEventListener('wheel', function (event) {
			if (event.deltaY > 0) {
				graph.zoomOut();
			} else {
				graph.zoomIn();
			}
			console.log(event.deltaY);
		})
	},
	methods: {
		test() {
			const xml = new ModelXmlSerializer(this.graph.getDataModel()).export();
			console.log(xml);
		}
	}
}
</script>


<style lang="scss">
.graphEditorContainer {
	width: 100vw;
	height: 100vh;
	padding: 0px;
	margin: 0px;
}

.graphContainer {
	width: 90vw;
	height: 90vh;
	border: solid 1px #00ff00;
}
.toolBar {
	position: absolute;
	top: 10px;
	left: 10px;
	transform-origin: 0px 0px;
	transform: scale(2);
	backdrop-filter: blur(6px);
	background: rgba(172, 172, 172, 0.533);
}
</style>
