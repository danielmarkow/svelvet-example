<script lang="ts">
	import { Anchor, Node, Svelvet } from 'svelvet';

	type Node = {
		id: number;
		position: {
			x: number;
			y: number;
		};
		label: string;
		dimensions: {
			width: number;
			height: number;
		};
		anchors: {
			id: string;
			connections: never[];
			output: boolean;
			dynamic: boolean;
		}[];
	};

	const CANVASHEIGHT = 500;
	const CANVESWIDTH = 1000;

	let nodeCounter = 1;
	let creationMode: 'lr' | 'rl' = 'lr';

	// form
	let newLabel = '';
	let editNode: Node;
  let showEditNode = false;

	let initialNodes = [
		{
			id: 1, // gets transformed to id string 'N-1', 0 is not supported
			position: { x: 10, y: 10 },
			label: 'Output Node',
			dimensions: { width: 175, height: 40 },
			anchors: [
				{
					id: 'anchor1',
					connections: [],
					output: true,
					dynamic: true
				}
			]
		}
	];

	function addNode() {
		const newNodeDimensions = { width: 175, height: 40 };
		const lastNodePostion = initialNodes.filter((node) => node.id === nodeCounter)[0].position;

		if (creationMode === 'lr') {
			let newNodePosition = { x: lastNodePostion.x + 225, y: lastNodePostion.y };

			// right side boundary
			if (newNodePosition.x + newNodeDimensions.width > CANVESWIDTH) {
				newNodePosition = { x: lastNodePostion.x, y: lastNodePostion.y + 100 };
				creationMode = 'rl';
			}
			initialNodes = [
				...initialNodes,
				{
					id: nodeCounter + 1,
					position: newNodePosition,
					label: newLabel,
					dimensions: newNodeDimensions,
					anchors: []
				}
			];
		}

		if (creationMode === 'rl') {
			let newNodePosition = { x: lastNodePostion.x - 225, y: lastNodePostion.y };

			// left side boundary
			if (newNodePosition.x + newNodeDimensions.width < 0) {
				newNodePosition = { x: lastNodePostion.x, y: lastNodePostion.y + 100 };
				creationMode = 'lr';
			}
			initialNodes = [
				...initialNodes,
				{
					id: nodeCounter + 1,
					position: newNodePosition,
					label: newLabel,
					dimensions: newNodeDimensions,
					anchors: []
				}
			];
		}

		newLabel = '';
		nodeCounter += 1;
	}

	function setEditNode(nodeId: number) {
		const nodeToEdit = initialNodes.find((n) => n.id === nodeId);
		if (nodeToEdit) {
			editNode = nodeToEdit;
		}
    showEditNode = true;
	}

	function saveNode(editNodeId: number) {
    initialNodes = [...initialNodes.filter(n => n.id !== editNodeId), editNode];
    showEditNode = false;
  }
</script>

<Svelvet height={CANVASHEIGHT} width={CANVESWIDTH}>
	{#each initialNodes as initialNode}
		<Node
			id={initialNode.id}
			position={initialNode.position}
			dimensions={initialNode.dimensions}
			useDefaults
			on:nodeClicked={() => setEditNode(initialNode.id)}
		>
			<p>{initialNode.label}</p>
			{#each initialNode.anchors as anchor}
				<Anchor
					id={anchor.id}
					output={anchor.output}
					input={!anchor.output}
					dynamic={anchor.dynamic}
					connections={anchor.connections}
				/>
			{/each}
		</Node>
	{/each}
</Svelvet>
<div style="padding: 1rem" />
<p>create a new note</p>
<input type="text" bind:value={newLabel} placeholder="label" />
<button type="button" on:click={() => addNode()}>add</button>
<br />
<p>edit note</p>
{#if typeof editNode !== "undefined" && showEditNode}
<input type="text" bind:value={editNode.label} />
<button type="button" on:click={() => saveNode(editNode.id)}>save</button>
{:else}
<p style="color: gray;">click on node to edit</p>
{/if}