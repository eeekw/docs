# buildChunkGraph

## buildChunkGraph

```js
const buildChunkGraph = (compilation, inputEntrypointsAndModules) => {
	const logger = compilation.getLogger("webpack.buildChunkGraph");

	// SHARED STATE

	/** @type {Map<AsyncDependenciesBlock, BlockChunkGroupConnection[]>} */
	const blockConnections = new Map();

	/** @type {Set<ChunkGroup>} */
	const allCreatedChunkGroups = new Set();

	/** @type {Map<ChunkGroup, ChunkGroupInfo>} */
	const chunkGroupInfoMap = new Map();

	/** @type {Set<DependenciesBlock>} */
	const blocksWithNestedBlocks = new Set();

	// PART ONE

	logger.time("visitModules");
  //  关联所有chunk与module
	visitModules(
		logger,
		compilation,
		inputEntrypointsAndModules,
		chunkGroupInfoMap,
		blockConnections,
		blocksWithNestedBlocks,
		allCreatedChunkGroups
	);
	logger.timeEnd("visitModules");

	// PART TWO

	logger.time("connectChunkGroups");
  // 连接block与chunkGroup
	connectChunkGroups(
		compilation,
		blocksWithNestedBlocks,
		blockConnections,
		chunkGroupInfoMap
	);
	logger.timeEnd("connectChunkGroups");
  // 为每个chunk设置runtime名字
	for (const [chunkGroup, chunkGroupInfo] of chunkGroupInfoMap) {
		for (const chunk of chunkGroup.chunks)
      // 合并chunk.runtime，多入口如果同时引用同一个chunk，这个chunk的runtime保存两个入口的runtime名字
			chunk.runtime = mergeRuntime(chunk.runtime, chunkGroupInfo.runtime);
	}

	// Cleanup work

	logger.time("cleanup");
	cleanupUnconnectedGroups(compilation, allCreatedChunkGroups);
	logger.timeEnd("cleanup");
}
```

## extractBlockModulesMap

获取所有模块的依赖的关联信息
```js
const extractBlockModulesMap = compilation => {
	const { moduleGraph } = compilation;

	/** @type {Map<DependenciesBlock, Map<Module, ModuleGraphConnection[]>>} */
	const blockModulesMap = new Map();

	const blockQueue = new Set();
  // 遍历所有的module
	for (const module of compilation.modules) {
		/** @type {WeakMap<Dependency, ModuleGraphConnection>} */
		let moduleMap;
    // 遍历module的outgoingConnections，里面主要包含依赖与模块的关联信息的集合
		for (const connection of moduleGraph.getOutgoingConnections(module)) {
			const d = connection.dependency;
			// We skip connections without dependency
			if (!d) continue;
			const m = connection.module;
			// We skip connections without Module pointer
			if (!m) continue;
			// We skip weak connections
			if (connection.weak) continue;
			const state = connection.getActiveState(undefined);
			// We skip inactive connections
			if (state === false) continue;
			// Store Dependency to Module mapping in local map
			// to allow to access it faster compared to
			// moduleGraph.getConnection()
			if (moduleMap === undefined) {
				moduleMap = new WeakMap();
			}
      // 只保存有用的connection供后面使用
			moduleMap.set(connection.dependency, connection);
		}

		blockQueue.clear();
		blockQueue.add(module);
    // 这里使用循环主要是为了处理子blocks里的模块
		for (const block of blockQueue) {
			let modules;

			if (moduleMap !== undefined && block.dependencies) {
				for (const dep of block.dependencies) {
          // 根据模块中的依赖取出对应的connection
					const connection = moduleMap.get(dep);
					if (connection !== undefined) {
						const { module } = connection;
						if (modules === undefined) {
							modules = new Map();
              // 保存模块的子模块信息
							blockModulesMap.set(block, modules);
						}
            // modules保存依赖对应模块的connection
						const old = modules.get(module);
						if (old !== undefined) {
							old.push(connection);
						} else {
							modules.set(module, [connection]);
						}
					}
				}
			}

			if (block.blocks) {
				for (const b of block.blocks) {
					blockQueue.add(b);
				}
			}
		}
	}

	return blockModulesMap;
}
```

## visitModules

```js
const visitModules = (
	logger,
	compilation,
	inputEntrypointsAndModules,
	chunkGroupInfoMap,
	blockConnections,
	blocksWithNestedBlocks,
	allCreatedChunkGroups
) => {
	const { moduleGraph, chunkGraph } = compilation;

	logger.time("visitModules: prepare");
  // 返回模块的依赖关联信息
	const blockModulesMap = extractBlockModulesMap(compilation);

	let statProcessedQueueItems = 0;
	let statProcessedBlocks = 0;
	let statConnectedChunkGroups = 0;
	let statProcessedChunkGroupsForMerging = 0;
	let statMergedAvailableModuleSets = 0;
	let statForkedAvailableModules = 0;
	let statForkedAvailableModulesCount = 0;
	let statForkedAvailableModulesCountPlus = 0;
	let statForkedMergedModulesCount = 0;
	let statForkedMergedModulesCountPlus = 0;
	let statForkedResultModulesCount = 0;
	let statChunkGroupInfoUpdated = 0;
	let statChildChunkGroupsReconnected = 0;

	let nextChunkGroupIndex = 0;
	let nextFreeModulePreOrderIndex = 0;
	let nextFreeModulePostOrderIndex = 0;

	/** @type {Map<DependenciesBlock, ChunkGroupInfo>} */
	const blockChunkGroups = new Map();

	/** @type {Map<string, ChunkGroupInfo>} */
	const namedChunkGroups = new Map();

	/** @type {Map<string, ChunkGroupInfo>} */
	const namedAsyncEntrypoints = new Map();

	const ADD_AND_ENTER_ENTRY_MODULE = 0;
	const ADD_AND_ENTER_MODULE = 1;
	const ENTER_MODULE = 2;
	const PROCESS_BLOCK = 3;
	const PROCESS_ENTRY_BLOCK = 4;
	const LEAVE_MODULE = 5;

	/** @type {QueueItem[]} */
	let queue = [];

	/** @type {Map<ChunkGroupInfo, Set<ChunkGroupInfo>>} */
	const queueConnect = new Map();
	/** @type {Set<ChunkGroupInfo>} */
	const chunkGroupsForCombining = new Set();

	// Fill queue with entrypoint modules
	// Create ChunkGroupInfo for entrypoints
  // 遍历入口
	for (const [chunkGroup, modules] of inputEntrypointsAndModules) {
    // 获取runtime chunk的名字，如果没有返回入口名
		const runtime = getEntryRuntime(
			compilation,
			chunkGroup.name,
			chunkGroup.options
		);
		/** @type {ChunkGroupInfo} */
		const chunkGroupInfo = {
			chunkGroup,
			runtime,
			minAvailableModules: undefined,
			minAvailableModulesOwned: false,
			availableModulesToBeMerged: [],
			skippedItems: undefined,
			resultingAvailableModules: undefined,
			children: undefined,
			availableSources: undefined,
			availableChildren: undefined,
			preOrderIndex: 0,
			postOrderIndex: 0
		};
		chunkGroup.index = nextChunkGroupIndex++;
    // 设置dependOn　chunkGroup才会有父子
		if (chunkGroup.getNumberOfParents() > 0) {
			// ...
		} else {
			// The application may start here: We start with an empty list of available modules
			chunkGroupInfo.minAvailableModules = EMPTY_SET;
      // 获取入口chunk
			const chunk = chunkGroup.getEntrypointChunk();
      // 遍历入口模块，每个入口模块组成的对象都放入队列中
			for (const module of modules) {
				queue.push({
					action: ADD_AND_ENTER_MODULE,
					block: module,
					module,
					chunk,
					chunkGroup,
					chunkGroupInfo
				});
			}
		}
		chunkGroupInfoMap.set(chunkGroup, chunkGroupInfo);
		if (chunkGroup.name) {
			namedChunkGroups.set(chunkGroup.name, chunkGroupInfo);
		}
	}
	// pop() is used to read from the queue
	// so it need to be reversed to be iterated in
	// correct order
  // 反转队列，使用pop()获取队列时按加入的顺序
	queue.reverse();

	/** @type {Set<ChunkGroupInfo>} */
	const outdatedChunkGroupInfo = new Set();
	/** @type {Set<ChunkGroupInfo>} */
	const chunkGroupsForMerging = new Set();
	/** @type {QueueItem[]} */
	let queueDelayed = [];

	logger.timeEnd("visitModules: prepare");

	/** @type {[Module, ModuleGraphConnection[]][]} */
	const skipConnectionBuffer = [];
	/** @type {Module[]} */
	const skipBuffer = [];
	/** @type {QueueItem[]} */
	const queueBuffer = [];

	/** @type {Module} */
	let module;
	/** @type {Chunk} */
	let chunk;
	/** @type {ChunkGroup} */
	let chunkGroup;
	/** @type {DependenciesBlock} */
	let block;
	/** @type {ChunkGroupInfo} */
	let chunkGroupInfo;

	// For each async Block in graph
	/**
	 * @param {AsyncDependenciesBlock} b iterating over each Async DepBlock
	 * @returns {void}
	 */
	const iteratorBlock = b => {
		// 1. We create a chunk group with single chunk in it for this Block
		// but only once (blockChunkGroups map)
		let cgi = blockChunkGroups.get(b);
		/** @type {ChunkGroup} */
		let c;
		/** @type {Entrypoint} */
		let entrypoint;
		const entryOptions = b.groupOptions && b.groupOptions.entryOptions;
		if (cgi === undefined) {
			const chunkName = (b.groupOptions && b.groupOptions.name) || b.chunkName;
			if (entryOptions) {
				cgi = namedAsyncEntrypoints.get(chunkName);
				if (!cgi) {
					entrypoint = compilation.addAsyncEntrypoint(
						entryOptions,
						module,
						b.loc,
						b.request
					);
					entrypoint.index = nextChunkGroupIndex++;
					cgi = {
						chunkGroup: entrypoint,
						runtime: entrypoint.options.runtime || entrypoint.name,
						minAvailableModules: EMPTY_SET,
						minAvailableModulesOwned: false,
						availableModulesToBeMerged: [],
						skippedItems: undefined,
						resultingAvailableModules: undefined,
						children: undefined,
						availableSources: undefined,
						availableChildren: undefined,
						preOrderIndex: 0,
						postOrderIndex: 0
					};
					chunkGroupInfoMap.set(entrypoint, cgi);

					chunkGraph.connectBlockAndChunkGroup(b, entrypoint);
					if (chunkName) {
						namedAsyncEntrypoints.set(chunkName, cgi);
					}
				} else {
					entrypoint = /** @type {Entrypoint} */ (cgi.chunkGroup);
					// TODO merge entryOptions
					entrypoint.addOrigin(module, b.loc, b.request);
					chunkGraph.connectBlockAndChunkGroup(b, entrypoint);
				}

				// 2. We enqueue the DependenciesBlock for traversal
				queueDelayed.push({
					action: PROCESS_ENTRY_BLOCK,
					block: b,
					module: module,
					chunk: entrypoint.chunks[0],
					chunkGroup: entrypoint,
					chunkGroupInfo: cgi
				});
			} else {
				cgi = namedChunkGroups.get(chunkName);
				if (!cgi) {
					c = compilation.addChunkInGroup(
						b.groupOptions || b.chunkName,
						module,
						b.loc,
						b.request
					);
					c.index = nextChunkGroupIndex++;
					cgi = {
						chunkGroup: c,
						runtime: chunkGroupInfo.runtime,
						minAvailableModules: undefined,
						minAvailableModulesOwned: undefined,
						availableModulesToBeMerged: [],
						skippedItems: undefined,
						resultingAvailableModules: undefined,
						children: undefined,
						availableSources: undefined,
						availableChildren: undefined,
						preOrderIndex: 0,
						postOrderIndex: 0
					};
					allCreatedChunkGroups.add(c);
					chunkGroupInfoMap.set(c, cgi);
					if (chunkName) {
						namedChunkGroups.set(chunkName, cgi);
					}
				} else {
					c = cgi.chunkGroup;
					if (c.isInitial()) {
						compilation.errors.push(
							new AsyncDependencyToInitialChunkError(chunkName, module, b.loc)
						);
						c = chunkGroup;
					}
					c.addOptions(b.groupOptions);
					c.addOrigin(module, b.loc, b.request);
				}
				blockConnections.set(b, []);
			}
			blockChunkGroups.set(b, cgi);
		} else if (entryOptions) {
			entrypoint = /** @type {Entrypoint} */ (cgi.chunkGroup);
		} else {
			c = cgi.chunkGroup;
		}

		if (c !== undefined) {
			// 2. We store the connection for the block
			// to connect it later if needed
			blockConnections.get(b).push({
				originChunkGroupInfo: chunkGroupInfo,
				chunkGroup: c
			});

			// 3. We enqueue the chunk group info creation/updating
			let connectList = queueConnect.get(chunkGroupInfo);
			if (connectList === undefined) {
				connectList = new Set();
				queueConnect.set(chunkGroupInfo, connectList);
			}
			connectList.add(cgi);

			// TODO check if this really need to be done for each traversal
			// or if it is enough when it's queued when created
			// 4. We enqueue the DependenciesBlock for traversal
			queueDelayed.push({
				action: PROCESS_BLOCK,
				block: b,
				module: module,
				chunk: c.chunks[0],
				chunkGroup: c,
				chunkGroupInfo: cgi
			});
		} else {
			chunkGroupInfo.chunkGroup.addAsyncEntrypoint(entrypoint);
		}
	};

	/**
	 * @param {DependenciesBlock} block the block
	 * @returns {void}
	 */
	const processBlock = block => {
		statProcessedBlocks++;
		// get prepared block info
		const blockModules = blockModulesMap.get(block);

		if (blockModules !== undefined) {
			const { minAvailableModules, runtime } = chunkGroupInfo;
			// Buffer items because order need to be reversed to get indices correct
			// Traverse all referenced modules
			for (const entry of blockModules) {
				const [refModule, connections] = entry;
				if (chunkGraph.isModuleInChunk(refModule, chunk)) {
					// skip early if already connected
					continue;
				}
				const activeState = getActiveStateOfConnections(connections, runtime);
				if (activeState !== true) {
					skipConnectionBuffer.push(entry);
					if (activeState === false) continue;
				}
				if (
					activeState === true &&
					(minAvailableModules.has(refModule) ||
						minAvailableModules.plus.has(refModule))
				) {
					// already in parent chunks, skip it for now
					skipBuffer.push(refModule);
					continue;
				}
				// enqueue, then add and enter to be in the correct order
				// this is relevant with circular dependencies
				queueBuffer.push({
					action: activeState === true ? ADD_AND_ENTER_MODULE : PROCESS_BLOCK,
					block: refModule,
					module: refModule,
					chunk,
					chunkGroup,
					chunkGroupInfo
				});
			}
			// Add buffered items in reverse order
			if (skipConnectionBuffer.length > 0) {
				let { skippedModuleConnections } = chunkGroupInfo;
				if (skippedModuleConnections === undefined) {
					chunkGroupInfo.skippedModuleConnections = skippedModuleConnections = new Set();
				}
				for (let i = skipConnectionBuffer.length - 1; i >= 0; i--) {
					skippedModuleConnections.add(skipConnectionBuffer[i]);
				}
				skipConnectionBuffer.length = 0;
			}
			if (skipBuffer.length > 0) {
				let { skippedItems } = chunkGroupInfo;
				if (skippedItems === undefined) {
					chunkGroupInfo.skippedItems = skippedItems = new Set();
				}
				for (let i = skipBuffer.length - 1; i >= 0; i--) {
					skippedItems.add(skipBuffer[i]);
				}
				skipBuffer.length = 0;
			}
			if (queueBuffer.length > 0) {
				for (let i = queueBuffer.length - 1; i >= 0; i--) {
					queue.push(queueBuffer[i]);
				}
				queueBuffer.length = 0;
			}
		}

		// Traverse all Blocks
		for (const b of block.blocks) {
			iteratorBlock(b);
		}

		if (block.blocks.length > 0 && module !== block) {
			blocksWithNestedBlocks.add(block);
		}
	};

	/**
	 * @param {DependenciesBlock} block the block
	 * @returns {void}
	 */
	const processEntryBlock = block => {
		statProcessedBlocks++;
		// get prepared block info
		const blockModules = blockModulesMap.get(block);

		if (blockModules !== undefined) {
			// Traverse all referenced modules
			for (const [refModule, connections] of blockModules) {
				const activeState = getActiveStateOfConnections(connections, undefined);
				// enqueue, then add and enter to be in the correct order
				// this is relevant with circular dependencies
				queueBuffer.push({
					action:
						activeState === true ? ADD_AND_ENTER_ENTRY_MODULE : PROCESS_BLOCK,
					block: refModule,
					module: refModule,
					chunk,
					chunkGroup,
					chunkGroupInfo
				});
			}
			// Add buffered items in reverse order
			if (queueBuffer.length > 0) {
				for (let i = queueBuffer.length - 1; i >= 0; i--) {
					queue.push(queueBuffer[i]);
				}
				queueBuffer.length = 0;
			}
		}

		// Traverse all Blocks
		for (const b of block.blocks) {
			iteratorBlock(b);
		}

		if (block.blocks.length > 0 && module !== block) {
			blocksWithNestedBlocks.add(block);
		}
	};

	const processQueue = () => {
		while (queue.length) {
			statProcessedQueueItems++;
			const queueItem = queue.pop();
			module = queueItem.module;
			block = queueItem.block;
			chunk = queueItem.chunk;// 入口chunk
			chunkGroup = queueItem.chunkGroup;
			chunkGroupInfo = queueItem.chunkGroupInfo;

			switch (queueItem.action) {
				case ADD_AND_ENTER_ENTRY_MODULE:
					chunkGraph.connectChunkAndEntryModule(
						chunk,
						module,
						/** @type {Entrypoint} */ (chunkGroup)
					);
				// fallthrough
				case ADD_AND_ENTER_MODULE: {// 入口模块会进入这个条件
        // 是否已经处理
					if (chunkGraph.isModuleInChunk(module, chunk)) {
						// already connected, skip it
						break;
					}
					// We connect Module and Chunk
          // 连接module与chunk
					chunkGraph.connectChunkAndModule(chunk, module);
				}
				// fallthrough
				case ENTER_MODULE: {
          // ...
				}
				// fallthrough
				case PROCESS_BLOCK: {
          // 从blockModulesMap中取出模块的依赖信息，把子模块相关信息加入队列中
					processBlock(block);
					break;
				}
				case PROCESS_ENTRY_BLOCK: {
					processEntryBlock(block);
					break;
				}
				case LEAVE_MODULE: {
          // ...
					break;
				}
			}
		}
	};

	// Iterative traversal of the Module graph
	// Recursive would be simpler to write but could result in Stack Overflows
  // 遍历队列，递归调用会导致栈溢出
	while (queue.length || queueConnect.size) {
		processQueue();

		// Run queueDelayed when all items of the queue are processed
		// This is important to get the global indexing correct
		// Async blocks should be processed after all sync blocks are processed
		if (queue.length === 0) {
			const tempQueue = queue;
			queue = queueDelayed.reverse();
			queueDelayed = tempQueue;
		}
	}
}
```