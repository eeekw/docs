# HarmonyImportSpecifierDependencyTemplate

## apply

因为webpack对替换了导入导出方法，我们的导出是绑定到一个对象上，所以导入后还需要从对象上取出。这里需要生成此用途的代码
```js
apply(dependency, source, templateContext) {
		const dep = /** @type {HarmonyImportSpecifierDependency} */ (dependency);
		const {
			moduleGraph,
			module,
			runtime,
			concatenationScope
		} = templateContext;
		const connection = moduleGraph.getConnection(dep);
		// Skip rendering depending when dependency is conditional
		if (connection && !connection.isTargetActive(runtime)) return;

		const ids = dep.getIds(moduleGraph);

		super.apply(dependency, source, templateContext);

		const {
			runtimeTemplate,
			initFragments,
			runtimeRequirements
		} = templateContext;
    // 因为导出经过了特殊处理，与module.export类似。所以引入模块后还需要拿到相应的对象，例如require后的对象上保存着我们导出的对象
		exportExpr = runtimeTemplate.exportFromImport({
			moduleGraph,
			module: moduleGraph.getModule(dep),
			request: dep.request,
			exportName: ids,
			originModule: module,
			asiSafe: dep.shorthand ? true : dep.asiSafe,
			isCall: dep.call,
			callContext: !dep.directImport,
			defaultInterop: true,
			importVar: dep.getImportVar(moduleGraph),
			initFragments,
			runtime,
			runtimeRequirements
		});
    // 替换代码片段
		if (dep.shorthand) {
			source.insert(dep.range[1], `: ${exportExpr}`);
		} else {
			source.replace(dep.range[0], dep.range[1] - 1, exportExpr);
		}
	}
```