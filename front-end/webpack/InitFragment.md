# InitFragment

## addToSource

```js
static addToSource(source, initFragments, generateContext) {
		if (initFragments.length > 0) {
			// Sort fragments by position. If 2 fragments have the same position,
			// use their index.
			const sortedFragments = initFragments
				.map(extractFragmentIndex)
				.sort(sortFragmentWithIndex);

			// Deduplicate fragments. If a fragment has no key, it is always included.
      // 合并有相同key的fragment
			const keyedFragments = new Map();
			for (const [fragment] of sortedFragments) {
				if (typeof fragment.merge === "function") {
					const oldValue = keyedFragments.get(fragment.key);
					if (oldValue !== undefined) {
						keyedFragments.set(
							fragment.key || Symbol(),
							fragment.merge(oldValue)
						);
						continue;
					}
				}
				keyedFragments.set(fragment.key || Symbol(), fragment);
			}
      // 用于存储最终生成的代码
			const concatSource = new ConcatSource();
			const endContents = [];
      // 遍历前面由依赖生成的所有的代码片段对象
			for (const fragment of keyedFragments.values()) {
        // 把引入模块的代码添加到源代码的头部
				concatSource.add(fragment.getContent(generateContext));
				const endContent = fragment.getEndContent(generateContext);
				if (endContent) {
					endContents.push(endContent);
				}
			}
      // 然后再把源代码添加到后面
			concatSource.add(source);
			for (const content of endContents.reverse()) {
				concatSource.add(content);
			}
			return concatSource;
		} else {
			return source;
		}
	}
```