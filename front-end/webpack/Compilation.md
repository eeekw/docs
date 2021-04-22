# Compilation

## seal

```js
seal(callback) {
	const chunkGraph = new ChunkGraph(this.moduleGraph);
	this.chunkGraph = chunkGraph;
	for (const module of this.modules) {
		ChunkGraph.setChunkGraphForModule(module, chunkGraph);
	}
	/** @type {Map<Entrypoint, Module[]>} */
	const chunkGraphInit = new Map();
  // 遍历入口，多页面应用入口集合中包括多个入口
  // 此处的遍历主要用于生成入口chunk与入口chunkGroup
	for (const [name, { dependencies, includeDependencies, options }] of this
		.entries) {
    // 根据入口创建Chunk
		const chunk = this.addChunk(name);
		if (options.filename) {
			chunk.filenameTemplate = options.filename;
		}
    // 创建入口ChunkGroup
		const entrypoint = new Entrypoint(options);
    // 设置runtime chunk
		if (!options.dependOn && !options.runtime) {
			entrypoint.setRuntimeChunk(chunk);
		}
    // 设置入口chunk
		entrypoint.setEntrypointChunk(chunk);
		this.namedChunkGroups.set(name, entrypoint);
		this.entrypoints.set(name, entrypoint);
		this.chunkGroups.push(entrypoint);
    // 连接入口chunk group 与　chunk
		connectChunkGroupAndChunk(entrypoint, chunk);
    // 遍历入口的依赖，一个入口可以有多个依赖对象
		for (const dep of [...this.globalEntry.dependencies, ...dependencies]) {
			entrypoint.addOrigin(null, { name }, /** @type {any} */ (dep).request);
      // 从moduleGraph中取出依赖对象对应的module(在此之前，会根据依赖创建相应的module对象)
			const module = this.moduleGraph.getModule(dep);
			if (module) {
        // 在chunkGraph中连接入口相关的chunk, module与entrypoint
				chunkGraph.connectChunkAndEntryModule(chunk, module, entrypoint);
        // 给module添加depth，表示module在依赖图中的深度
				this.assignDepth(module);
        // 保存每个入口对应的module，后面使用
				const modulesList = chunkGraphInit.get(entrypoint);
				if (modulesList === undefined) {
					chunkGraphInit.set(entrypoint, [module]);
				} else {
					modulesList.push(module);
				}
			}
		}
	}
	const runtimeChunks = new Set();
  // 此处的遍历主要处理dependOn与runtime选项
	outer: for (const [
		name,
		{
			options: { dependOn, runtime }
		}
	] of this.entries) {
		if (dependOn && runtime) {
      // ...
		}
		if (dependOn) {
			// ...

		} else if (runtime) {
      // 如果入口中有指定runtime，创建runtime chunk，并更新入口chunk group中的runtime chunk
			const entry = this.entrypoints.get(name);
			let chunk = this.namedChunks.get(runtime);
			if (chunk) {
        // ...
					continue;
				}
			} else {
				chunk = this.addChunk(runtime);
				chunk.preventIntegration = true;
				runtimeChunks.add(chunk);
			}
      // 连接入口chunk group与runtime chunk，近似于上面的connectChunkGroupAndChunk
			entry.unshiftChunk(chunk);
			chunk.addGroup(entry);
      // 设置runtime chunk
			entry.setRuntimeChunk(chunk);
		}
	}
  // 连接所有modules, blocks与chunks，并给chunks设置runtime名字
	buildChunkGraph(this, chunkGraphInit);

  // optimizeChunks: 此hook会执行SplitChunksPlugin
	while (this.hooks.optimizeChunks.call(this.chunks, this.chunkGroups)) {
		/* empty */
	}
	this.hooks.optimizeTree.callAsync(this.chunks, this.modules, err => {
		this.hooks.optimizeChunkModules.callAsync(
			this.chunks,
			this.modules,
			err => {
				// 设置runtime id
				this.assignRuntimeIds();
        // 排序
				this.sortItemsWithChunkIds();
        // 给每个module创建hash值，如果module所在chunk具有多个runtime，那么同一个module会根据不同的runtime生成不同的hash
				this.createModuleHashes();
        // 为每个模块生成代码
				this.codeGeneration(err => {
          // 处理运行时需要的代码
					this.processRuntimeRequirements();
          // 获取未处理的jobs
					const codeGenerationJobs = this.createHash();
          // 处理jobs，同codeGeneration
					this._runCodeGenerationJobs(codeGenerationJobs, err => {
            // 清空上次编译需要输出的文件信息
						this.clearAssets();
						this.hooks.beforeModuleAssets.call();
            // 输出模块中通过emitFile()发出的文件信息
						this.createModuleAssets();
						this.logger.timeEnd("module assets");
						const cont = () => {
							this.logger.time("process assets");
							this.hooks.processAssets.callAsync(this.assets, err => {
								return this.hooks.afterSeal.callAsync(err => {
									this.fileSystemInfo.logStatistics();
									callback();
								});
							});
						};
						this.logger.time("create chunk assets");
						if (this.hooks.shouldGenerateChunkAssets.call() !== false) {
							this.hooks.beforeChunkAssets.call();
							this.createChunkAssets(err => {
								cont();
							});
						} else {
							this.logger.timeEnd("create chunk assets");
							cont();
						}
					});
				});
			}
		);
	});
}
```

## codeGeneration

codeGeneration主要遍历modules，执行module的codeGeneration方法，最终的结果保存在codeGenerationResults中
```js
this.codeGenerationResults = new CodeGenerationResults()
const results = this.codeGenerationResults;

const jobs = [];
// 遍历modules，每个module根据runtime会创建不小于一个的job
for (const module of this.modules) {
	const runtimes = chunkGraph.getModuleRuntimes(module);
	if (runtimes.size === 1) {
		for (const runtime of runtimes) {
			const hash = chunkGraph.getModuleHash(module, runtime);
			jobs.push({ module, hash, runtime, runtimes: [runtime] });
		}
	} else if (runtimes.size > 1) {
		/** @type {Map<string, { runtimes: RuntimeSpec[] }>} */
		const map = new Map();
		for (const runtime of runtimes) {
			const hash = chunkGraph.getModuleHash(module, runtime);
			const job = map.get(hash);
			if (job === undefined) {
				const newJob = { module, hash, runtime, runtimes: [runtime] };
				jobs.push(newJob);
				map.set(hash, newJob);
			} else {
				job.runtimes.push(runtime);
			}
		}
	}
}
// 遍历上面创建的jobs执行以下行为
// 生成模块的代码对象，用于生成最终的代码
result = module.codeGeneration({
	chunkGraph,
	moduleGraph,
	dependencyTemplates,
	runtimeTemplate,
	runtime
});
for (const runtime of runtimes) {
	results.add(module, runtime, result);
}
```

## processRuntimeRequirements

```js
processRuntimeRequirements() {
	const { chunkGraph } = this;

	const additionalModuleRuntimeRequirements = this.hooks
		.additionalModuleRuntimeRequirements;
	const runtimeRequirementInModule = this.hooks.runtimeRequirementInModule;
  // 把runtimeRequirements保存到chunkGraph的每个模块中，暴露给hooks
	for (const module of this.modules) {
		if (chunkGraph.getNumberOfModuleChunks(module) > 0) {
			for (const runtime of chunkGraph.getModuleRuntimes(module)) {
				let set;
				const runtimeRequirements = this.codeGenerationResults.getRuntimeRequirements(
					module,
					runtime
				);
				if (runtimeRequirements && runtimeRequirements.size > 0) {
					set = new Set(runtimeRequirements);
				} else if (additionalModuleRuntimeRequirements.isUsed()) {
					set = new Set();
				} else {
					continue;
				}
        additionalModuleRuntimeRequirements.call(module, set);

				for (const r of set) {
					const hook = runtimeRequirementInModule.get(r);
					if (hook !== undefined) hook.call(module, set);
				}
				chunkGraph.addModuleRuntimeRequirements(module, runtime, set);
			}
		}
	}
  // 以chunk为单位提取runtimeRequirements保存到chunkGraph，暴露给hooks
	for (const chunk of this.chunks) {
		const set = new Set();
		for (const module of chunkGraph.getChunkModulesIterable(chunk)) {
			const runtimeRequirements = chunkGraph.getModuleRuntimeRequirements(
				module,
				chunk.runtime
			);
			for (const r of runtimeRequirements) set.add(r);
		}
		this.hooks.additionalChunkRuntimeRequirements.call(chunk, set);

		for (const r of set) {
			this.hooks.runtimeRequirementInChunk.for(r).call(chunk, set);
		}

		chunkGraph.addChunkRuntimeRequirements(chunk, set);
	}

    // 获取入口runtime chunk
		/** @type {Set<Chunk>} */
		const treeEntries = new Set();
		for (const ep of this.entrypoints.values()) {
			const chunk = ep.getRuntimeChunk();
			if (chunk) treeEntries.add(chunk);
		}
    // 获取异步入口runtime chunk
		for (const ep of this.asyncEntrypoints) {
			const chunk = ep.getRuntimeChunk();
			if (chunk) treeEntries.add(chunk);
		}

		for (const treeEntry of treeEntries) {
			const set = new Set();
      // 获取入口runtime chunk属于同一个chunkGroup（包括子chunkGroup）的所有chunk的runtimeRequirements保存到chunkGraph，暴露给hooks
			for (const chunk of treeEntry.getAllReferencedChunks()) {
				const runtimeRequirements = chunkGraph.getChunkRuntimeRequirements(
					chunk
				);
				for (const r of runtimeRequirements) set.add(r);
			}

			this.hooks.additionalTreeRuntimeRequirements.call(treeEntry, set);

			for (const r of set) {
				this.hooks.runtimeRequirementInTree.for(r).call(treeEntry, set);
			}

			chunkGraph.addTreeRuntimeRequirements(treeEntry, set);
		}
	}
```

## createHash

```js
createHash() {
		this.logger.time("hashing: initialize hash");
		const chunkGraph = this.chunkGraph;
		const runtimeTemplate = this.runtimeTemplate;
		const outputOptions = this.outputOptions;
		const hashFunction = outputOptions.hashFunction;
		const hashDigest = outputOptions.hashDigest;
		const hashDigestLength = outputOptions.hashDigestLength;
    // 创建hash对象
		const hash = createHash(hashFunction);
		this.logger.timeEnd("hashing: initialize hash");

		this.logger.time("hashing: sort chunks");
		/*
		 * all non-runtime chunks need to be hashes first,
		 * since runtime chunk might use their hashes.
		 * runtime chunks need to be hashed in the correct order
		 * since they may depend on each other (for async entrypoints).
		 * So we put all non-runtime chunks first and hash them in any order.
		 * And order runtime chunks according to referenced between each other.
		 * Chunks need to be in deterministic order since we add hashes to full chunk
		 * during these hashing.
		 */
    // 所以非runtime chunks需要先计算hash，因为runtime chunk可能需要它们的hash。runtime chunks需要按正确的顺序计算hash，因为它们可能依赖于其它chunks。所以我们首先计算非runtime chunks且没有顺序要求
    // 根据runtime chunks互相间的引用排序
    // chunks需要按一定的顺序因为
    // 把runtime chunks和非runtime chunks分开
		/** @type {Chunk[]} */
		const unorderedRuntimeChunks = [];
		/** @type {Chunk[]} */
		const otherChunks = [];
		for (const c of this.chunks) {
			if (c.hasRuntime()) {
				unorderedRuntimeChunks.push(c);
			} else {
				otherChunks.push(c);
			}
		}
		unorderedRuntimeChunks.sort(byId);
		otherChunks.sort(byId);

		this.logger.timeEnd("hashing: sort chunks");

		const fullHashChunks = new Set();
		/** @type {{module: Module, hash: string, runtime: RuntimeSpec, runtimes: RuntimeSpec[]}[]} */
		const codeGenerationJobs = [];
		/** @type {Map<string, Map<Module, {module: Module, hash: string, runtime: RuntimeSpec, runtimes: RuntimeSpec[]}>>} */
		const codeGenerationJobsMap = new Map();

		const processChunk = chunk => {
			// Last minute module hash generation for modules that depend on chunk hashes
      // 确保所有的module都生成了hash，同createModuleHashes
			this.logger.time("hashing: hash runtime modules");
			const runtime = chunk.runtime;
			for (const module of chunkGraph.getChunkModulesIterable(chunk)) {
				if (!chunkGraph.hasModuleHashes(module, runtime)) {
					const hash = this._createModuleHash(
						module,
						chunkGraph,
						runtime,
						hashFunction,
						runtimeTemplate,
						hashDigest,
						hashDigestLength
					);
					let hashMap = codeGenerationJobsMap.get(hash);
					if (hashMap) {
						const moduleJob = hashMap.get(module);
						if (moduleJob) {
							moduleJob.runtimes.push(runtime);
							continue;
						}
					} else {
						hashMap = new Map();
						codeGenerationJobsMap.set(hash, hashMap);
					}
					const job = {
						module,
						hash,
						runtime,
						runtimes: [runtime]
					};
					hashMap.set(module, job);
					codeGenerationJobs.push(job);
				}
			}
			// 创建hash实例
			const chunkHash = createHash(hashFunction);
      // 生成chunk的hash
			chunk.updateHash(chunkHash, chunkGraph);
			this.hooks.chunkHash.call(chunk, chunkHash, {
				chunkGraph,
				moduleGraph: this.moduleGraph,
				runtimeTemplate: this.runtimeTemplate
			});
      // 生成摘要
			const chunkHashDigest = /** @type {string} */ (chunkHash.digest(
				hashDigest
			));
			hash.update(chunkHashDigest);
			chunk.hash = chunkHashDigest;
      // 根据传入的长度截取
			chunk.renderedHash = chunk.hash.substr(0, hashDigestLength);
			const fullHashModules = chunkGraph.getChunkFullHashModulesIterable(
				chunk
			);
			if (fullHashModules) {
				fullHashChunks.add(chunk);
			} else {
        // JavascriptModulesPlugin应用了此hook，计算contentHash
				this.hooks.contentHash.call(chunk);
			}
		};
    // 计算非runtime chunks的hash
		otherChunks.forEach(processChunk);
    // 计算runtime chunks的hash
		for (const chunk of runtimeChunks) processChunk(chunk);

		return codeGenerationJobs;
	}
```

## createChunkAssets

根据chunks生成最终的文件信息
```js
createChunkAssets(callback) {
		asyncLib.forEach(
			this.chunks,
			(chunk, callback) => {
        // 这是主要触发renderManifest hook，JavascriptModulesPlugin使用了此hook返回了manifest数组
				let manifest;
				manifest = this.getRenderManifest({
					chunk,
					hash: this.hash,
					fullHash: this.fullHash,
					outputOptions,
					codeGenerationResults: this.codeGenerationResults,
					moduleTemplates: this.moduleTemplates,
					dependencyTemplates: this.dependencyTemplates,
					chunkGraph: this.chunkGraph,
					moduleGraph: this.moduleGraph,
					runtimeTemplate: this.runtimeTemplate
				});
        // 遍历manifest，执行render方法，返回最终需要输出的source对象
				asyncLib.forEach(
					manifest,
					(fileManifest, callback) => {
            
						// render the asset
            // 调用render方法，返回要输出的代码
						source = fileManifest.render();
            // 所有准备输出的文件保存在这里
						this.emitAsset(file, source, assetInfo);
					},
					callback
				);
			},
			callback
		);
	}
```