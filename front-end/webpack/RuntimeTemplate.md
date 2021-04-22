# RuntimeTemplate

## importStatement

生成运行时的引入模块的语句，替换原来的import
```js
importStatement({
		update,
		module,
		chunkGraph,
		request,
		importVar,
		originModule,
		weak,
		runtimeRequirements
	}) {
		const moduleId = this.moduleId({
			module,
			chunkGraph,
			request,
			weak
		});
		const optDeclaration = update ? "" : "var ";

		const exportsType = module.getExportsType(
			chunkGraph.moduleGraph,
			originModule.buildMeta.strictHarmonyModule
		);
    // 添加运行时需要全局注入的函数、对象等
		runtimeRequirements.add(RuntimeGlobals.require);
		const importContent = `/* harmony import */ ${optDeclaration}${importVar} = __webpack_require__(${moduleId});\n`;

		if (exportsType === "dynamic") {
			runtimeRequirements.add(RuntimeGlobals.compatGetDefaultExport);
			return [
				importContent,
				`/* harmony import */ ${optDeclaration}${importVar}_default = /*#__PURE__*/${RuntimeGlobals.compatGetDefaultExport}(${importVar});\n`
			];
		}
		return [importContent, ""];
	}
```