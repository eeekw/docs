# JavascriptGenerator

## generate

```js
generate(module, generateContext) {
  // 返回模块源码
	const originalSource = module.originalSource();
  // 
	const source = new ReplaceSource(originalSource);
	const initFragments = [];

	this.sourceModule(module, initFragments, source, generateContext);
  // 生成拼装的source对象
	return InitFragment.addToSource(source, initFragments, generateContext);
}
```

## sourceModule

遍历所有的依赖，根据依赖生成对应的代码片段保存到initFragments中
依赖包括导入和导出的语句
```js
sourceModule(module, initFragments, source, generateContext) {
	for (const dependency of module.dependencies) {
		this.sourceDependency(
			module,
			dependency,
			initFragments,
			source,
			generateContext
		);
	}

	if (module.presentationalDependencies !== undefined) {
		for (const dependency of module.presentationalDependencies) {
			this.sourceDependency(
				module,
				dependency,
				initFragments,
				source,
				generateContext
			);
		}
	}

	for (const childBlock of module.blocks) {
		this.sourceBlock(
			module,
			childBlock,
			initFragments,
			source,
			generateContext
		);
	}
}
```

## sourceDependency

通过依赖模板创建的代码片段对象保存在initFragments中
```js
sourceDependency(module, dependency, initFragments, source, generateContext) {
		const constructor = /** @type {new (...args: any[]) => Dependency} */ (dependency.constructor);
		const template = generateContext.dependencyTemplates.get(constructor);

		const templateContext = {
			runtimeTemplate: generateContext.runtimeTemplate,
			dependencyTemplates: generateContext.dependencyTemplates,
			moduleGraph: generateContext.moduleGraph,
			chunkGraph: generateContext.chunkGraph,
			module,
			runtime: generateContext.runtime,
			runtimeRequirements: generateContext.runtimeRequirements,
			concatenationScope: generateContext.concatenationScope,
			initFragments
		};
    // 调用依赖的对应的模块生成对应的InitFragment的子类
    // import会调用HarmonyImportDependencyTemplate生成引入模块的ConditionalInitFragment对象
		template.apply(dependency, source, templateContext);
	}
```