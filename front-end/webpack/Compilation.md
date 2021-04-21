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
  // 连接所有modules, blocks与chunks
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
        // 给每个module创建hash值
				this.createModuleHashes();
				this.codeGeneration(err => {
					this.processRuntimeRequirements();
					const codeGenerationJobs = this.createHash();
					this._runCodeGenerationJobs(codeGenerationJobs, err => {
						this.clearAssets();
						this.hooks.beforeModuleAssets.call();
						this.createModuleAssets();
						this.logger.timeEnd("module assets");
						const cont = () => {
							this.logger.time("process assets");
							this.hooks.processAssets.callAsync(this.assets, err => {
								if (err) {
									return callback(
										makeWebpackError(err, "Compilation.hooks.processAssets")
									);
								}
								this.hooks.afterProcessAssets.call(this.assets);
								this.logger.timeEnd("process assets");
								this.assets = soonFrozenObjectDeprecation(
									this.assets,
									"Compilation.assets",
									"DEP_WEBPACK_COMPILATION_ASSETS",
									`BREAKING CHANGE: No more changes should happen to Compilation.assets after sealing the Compilation.
Do changes to assets earlier, e. g. in Compilation.hooks.processAssets.
Make sure to select an appropriate stage from Compilation.PROCESS_ASSETS_STAGE_*.`
								);
								this.summarizeDependencies();
								if (shouldRecord) {
									this.hooks.record.call(this, this.records);
								}
								if (this.hooks.needAdditionalSeal.call()) {
									this.unseal();
									return this.seal(callback);
								}
								return this.hooks.afterSeal.callAsync(err => {
									if (err) {
										return callback(
											makeWebpackError(err, "Compilation.hooks.afterSeal")
										);
									}
									this.fileSystemInfo.logStatistics();
									callback();
								});
							});
						};
						this.logger.time("create chunk assets");
						if (this.hooks.shouldGenerateChunkAssets.call() !== false) {
							this.hooks.beforeChunkAssets.call();
							this.createChunkAssets(err => {
								this.logger.timeEnd("create chunk assets");
								if (err) {
									return callback(err);
								}
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
