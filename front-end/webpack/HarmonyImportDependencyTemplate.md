# HarmonyImportDependencyTemplate

## apply

```js
apply(dependency, source, templateContext) {
		const dep = /** @type {HarmonyImportDependency} */ (dependency);
		const { module, chunkGraph, moduleGraph, runtime } = templateContext;

		const connection = moduleGraph.getConnection(dep);

		const referencedModule = connection && connection.module;

		const moduleKey = referencedModule
			? referencedModule.identifier()
			: dep.request;
		const key = `harmony import ${moduleKey}`;

		const runtimeCondition = dep.weak
			? false
			: connection
			? filterRuntime(runtime, r => connection.isTargetActive(r))
			: true;
    // 返回引入模块的代码片段对象
		const importStatement = dep.getImportStatement(false, templateContext);
		if (templateContext.moduleGraph.isAsync(referencedModule)) {
			templateContext.initFragments.push(
				new ConditionalInitFragment(
					importStatement[0],
					InitFragment.STAGE_HARMONY_IMPORTS,
					dep.sourceOrder,
					key,
					runtimeCondition
				)
			);
			templateContext.initFragments.push(
				new AwaitDependenciesInitFragment(
					new Set([dep.getImportVar(templateContext.moduleGraph)])
				)
			);
			templateContext.initFragments.push(
				new ConditionalInitFragment(
					importStatement[1],
					InitFragment.STAGE_ASYNC_HARMONY_IMPORTS,
					dep.sourceOrder,
					key + " compat",
					runtimeCondition
				)
			);
		} else {
			templateContext.initFragments.push(
				new ConditionalInitFragment(
					importStatement[0] + importStatement[1],
					InitFragment.STAGE_HARMONY_IMPORTS,
					dep.sourceOrder,
					key,
					runtimeCondition
				)
			);
		}
	}
```