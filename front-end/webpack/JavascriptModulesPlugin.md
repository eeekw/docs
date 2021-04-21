# JavascriptModulesPlugin

## renderManifest

```js
compilation.hooks.renderManifest.tap(
	"JavascriptModulesPlugin",
	(result, options) => {
		const {
			hash,
			chunk,
			chunkGraph,
			moduleGraph,
			runtimeTemplate,
			dependencyTemplates,
			outputOptions,
			codeGenerationResults
		} = options;

		const hotUpdateChunk =
			chunk instanceof HotUpdateChunk ? chunk : null;

		let render;
		const filenameTemplate = JavascriptModulesPlugin.getChunkFilenameTemplate(
			chunk,
			outputOptions
		);
		if (hotUpdateChunk) {
			render = () =>
				this.renderChunk(
					{
						chunk,
						dependencyTemplates,
						runtimeTemplate,
						moduleGraph,
						chunkGraph,
						codeGenerationResults
					},
					hooks
				);
		} else if (chunk.hasRuntime()) {
			render = () =>
				this.renderMain(
					{
						hash,
						chunk,
						dependencyTemplates,
						runtimeTemplate,
						moduleGraph,
						chunkGraph,
						codeGenerationResults
					},
					hooks,
					compilation
				);
		} else {
			if (!chunkHasJs(chunk, chunkGraph)) {
				return result;
			}

			render = () =>
				this.renderChunk(
					{
						chunk,
						dependencyTemplates,
						runtimeTemplate,
						moduleGraph,
						chunkGraph,
						codeGenerationResults
					},
					hooks
				);
		}

		result.push({
			render,
			filenameTemplate,
			pathOptions: {
				hash,
				runtime: chunk.runtime,
				chunk,
				contentHashType: "javascript"
			},
			info: {
				javascriptModule: compilation.runtimeTemplate.isModule()
			},
			identifier: hotUpdateChunk
				? `hotupdatechunk${chunk.id}`
				: `chunk${chunk.id}`,
			hash: chunk.contentHash.javascript
		});

		return result;
	}
);
```

## 生成入口chunk的结果

```js
renderMain(renderContext, hooks, compilation) {
	const { chunk, chunkGraph, runtimeTemplate } = renderContext;
	const runtimeRequirements = chunkGraph.getTreeRuntimeRequirements(chunk);
	const iife = runtimeTemplate.isIIFE();
	const bootstrap = this.renderBootstrap(renderContext, hooks);
	const useSourceMap = hooks.useSourceMap.call(chunk, renderContext);
	const allModules = Array.from(
		chunkGraph.getOrderedChunkModulesIterableBySourceType(
			chunk,
			"javascript",
			compareModulesByIdentifier
		) || []
	);
	let inlinedModules;
	if (bootstrap.allowInlineStartup) {
		inlinedModules = new Set(chunkGraph.getChunkEntryModulesIterable(chunk));
	}
	let source = new ConcatSource();
	let prefix;
	if (iife) {
		if (runtimeTemplate.vsupportsArrowFunction()) {
			source.add("/******/ (() => { // webpackBootstrap\n");
		} else {
			source.add("/******/ (function() { // webpackBootstrap\n");
		}
		prefix = "/******/ \t";
	} else {
		prefix = "/******/ ";
	}
	let allStrict = false;
	if (allModules.every(m => m.buildInfo.strict)) {
		const strictBailout = hooks.strictRuntimeBailout.call(renderContext);
		if (strictBailout) {
			source.add(
				prefix +
					`// runtime can't be in strict mode because ${strictBailout}.\n`
			);
		} else {
			allStrict = true;
			source.add(prefix + '"use strict";\n');
		}
	}
	const chunkModules = Template.renderChunkModules(
		renderContext,
		inlinedModules
			? allModules.filter(m => !inlinedModules.has(m))
			: allModules,
		module =>
			this.renderModule(
				module,
				renderContext,
				hooks,
				allStrict ? "strict" : true
			),
		prefix
	);
	if (
		chunkModules ||
		runtimeRequirements.has(RuntimeGlobals.moduleFactories) ||
		runtimeRequirements.has(RuntimeGlobals.moduleFactoriesAddOnly)
	) {
		source.add(prefix + "var __webpack_modules__ = (");
		source.add(chunkModules || "{}");
		source.add(");\n");
		source.add(
			"/************************************************************************/\n"
		);
	}
	if (bootstrap.header.length > 0) {
		const header = Template.asString(bootstrap.header) + "\n";
		source.add(
			new PrefixSource(
				prefix,
				useSourceMap
					? new OriginalSource(header, "webpack/bootstrap")
					: new RawSource(header)
			)
		);
		source.add(
			"/************************************************************************/\n"
		);
	}
	const runtimeModules = renderContext.chunkGraph.getChunkRuntimeModulesInOrder(
		chunk
	);
	if (runtimeModules.length > 0) {
		source.add(
			new PrefixSource(
				prefix,
				Template.renderRuntimeModules(runtimeModules, renderContext)
			)
		);
		source.add(
			"/************************************************************************/\n"
		);
		// runtimeRuntimeModules calls codeGeneration
		for (const module of runtimeModules) {
			compilation.codeGeneratedModules.add(module);
		}
	}
	if (inlinedModules) {
		if (bootstrap.beforeStartup.length > 0) {
			const beforeStartup = Template.asString(bootstrap.beforeStartup) + "\n";
			source.add(
				new PrefixSource(
					prefix,
					useSourceMap
						? new OriginalSource(beforeStartup, "webpack/before-startup")
						: new RawSource(beforeStartup)
				)
			);
		}
		const lastInlinedModule = last(inlinedModules);
		const startupSource = new ConcatSource();
		startupSource.add(`var __webpack_exports__ = {};\n`);
		for (const m of inlinedModules) {
			const renderedModule = this.renderModule(
				m,
				renderContext,
				hooks,
				false
			);
			if (renderedModule) {
				const innerStrict = !allStrict && m.buildInfo.strict;
				const runtimeRequirements = chunkGraph.getModuleRuntimeRequirements(
					m,
					chunk.runtime
				);
				const exports = runtimeRequirements.has(RuntimeGlobals.exports);
				const webpackExports =
					exports && m.exportsArgument === "__webpack_exports__";
				let iife = innerStrict
					? "it need to be in strict mode."
					: inlinedModules.size > 1
					? // TODO check globals and top-level declarations of other entries and chunk modules
					  // to make a better decision
					  "it need to be isolated against other entry modules."
					: chunkModules
					? "it need to be isolated against other modules in the chunk."
					: exports && !webpackExports
					? `it uses a non-standard name for the exports (${m.exportsArgument}).`
					: hooks.embedInRuntimeBailout.call(m, renderContext);
				let footer;
				if (iife !== undefined) {
					startupSource.add(
						`// This entry need to be wrapped in an IIFE because ${iife}\n`
					);
					const arrow = runtimeTemplate.supportsArrowFunction();
					if (arrow) {
						startupSource.add("(() => {\n");
						footer = "\n})();\n\n";
					} else {
						startupSource.add("!function() {\n");
						footer = "\n}();\n";
					}
					if (innerStrict) startupSource.add('"use strict";\n');
				} else {
					footer = "\n";
				}
				if (exports) {
					if (m !== lastInlinedModule)
						startupSource.add(`var ${m.exportsArgument} = {};\n`);
					else if (m.exportsArgument !== "__webpack_exports__")
						startupSource.add(
							`var ${m.exportsArgument} = __webpack_exports__;\n`
						);
				}
				startupSource.add(renderedModule);
				startupSource.add(footer);
			}
		}
		if (runtimeRequirements.has(RuntimeGlobals.onChunksLoaded)) {
			startupSource.add(`${RuntimeGlobals.onChunksLoaded}();\n`);
		}
		source.add(
			hooks.renderStartup.call(startupSource, lastInlinedModule, {
				...renderContext,
				inlined: true
			})
		);
		if (bootstrap.afterStartup.length > 0) {
			const afterStartup = Template.asString(bootstrap.afterStartup) + "\n";
			source.add(
				new PrefixSource(
					prefix,
					useSourceMap
						? new OriginalSource(afterStartup, "webpack/after-startup")
						: new RawSource(afterStartup)
				)
			);
		}
	} else {
		const lastEntryModule = last(
			chunkGraph.getChunkEntryModulesIterable(chunk)
		);
		const toSource = useSourceMap
			? (content, name) =>
					new OriginalSource(Template.asString(content), name)
			: content => new RawSource(Template.asString(content));
		source.add(
			new PrefixSource(
				prefix,
				new ConcatSource(
					toSource(bootstrap.beforeStartup, "webpack/before-startup"),
					"\n",
					hooks.renderStartup.call(
						toSource(bootstrap.startup.concat(""), "webpack/startup"),
						lastEntryModule,
						{
							...renderContext,
							inlined: false
						}
					),
					toSource(bootstrap.afterStartup, "webpack/after-startup"),
					"\n"
				)
			)
		);
	}
	if (runtimeRequirements.has(RuntimeGlobals.returnExportsFromRuntime)) {
		source.add(`${prefix}return __webpack_exports__;\n`);
	}
	if (iife) {
		source.add("/******/ })()\n");
	}
	/** @type {Source} */
	let finalSource = tryRunOrWebpackError(
		() => hooks.renderMain.call(source, renderContext),
		"JavascriptModulesPlugin.getCompilationHooks().renderMain"
	);
	if (!finalSource) {
		throw new Error(
			"JavascriptModulesPlugin error: JavascriptModulesPlugin.getCompilationHooks().renderMain plugins should return something"
		);
	}
	finalSource = tryRunOrWebpackError(
		() => hooks.render.call(finalSource, renderContext),
		"JavascriptModulesPlugin.getCompilationHooks().render"
	);
	if (!finalSource) {
		throw new Error(
			"JavascriptModulesPlugin error: JavascriptModulesPlugin.getCompilationHooks().render plugins should return something"
		);
	}
	chunk.rendered = true;
	return iife ? new ConcatSource(finalSource, ";") : finalSource;
}
```