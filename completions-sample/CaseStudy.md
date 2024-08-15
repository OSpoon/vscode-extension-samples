# Completion Item Provider Sample

- 项目路径：vscode-extension-samples/completions-sample

支持在编辑器中提供补全功能（智能感知），需要使用到 `CompletionItemProvider` API。

## 使用概述：

定义一个 `CompletionItemProvider` 对象主要由三步操作：

1. 通过 `provideCompletionItems()` 方法回调的 `document` 和 `position` 参数提取编辑器中的文本内容；
2. 通过正则或其他方式验证提取到的文本内容是否符合要求；
3. 符合要求的情况下通过返回一组 `CompletionItem` 对象实现补全提示。

```tsx
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
	
	// 定义 CompletionItemProvider
	const provider: vscode.CompletionItemProvider = {
		provideCompletionItems(document: vscode.TextDocument, position: vscode.Position) {

			// 提取 & 匹配
			const linePrefix = document.lineAt(position).text.slice(0, position.character);
			if (!linePrefix.endsWith('console.')) {
				return undefined;
			}

			const items = [
				new vscode.CompletionItem('log', vscode.CompletionItemKind.Method),
			];
			
			// 返回 CompletionItem 数组
			return items;
		}
	};

	// 注册 CompletionItemProvider
	const dispose = vscode.languages.registerCompletionItemProvider('plaintext', provider, '.');
	context.subscriptions.push(dispose);
}
```

## 官方案例分析：

在 `src/extension.ts` 中提供了两个 provider 对象，provider1 主要是演示如何创建合适的 CompletionItem 的实例，而 provider2 则是演示如何提取和匹配文本内容。

### Sample Provider1：

1. 这个补全项用来插入一段 'Hello World!' 文本内容：

```tsx
const simpleCompletion = new vscode.CompletionItem('Hello World!');
```

1. 这个补全项的插入文本由 Snippet String 构成，选中此补全项后会再次提示由 `morning`,`afternoon` 和 `evening` 组成的补全项；Markdown String 还为补全项提供了丰富的文档展示：

```tsx
const snippetCompletion = new vscode.CompletionItem('Good part of the day');
snippetCompletion.insertText = new vscode.SnippetString('Good ${1|morning,afternoon,evening|}. It is ${1}, right?');
const docs: any = new vscode.MarkdownString("Inserts a snippet that lets you select [link](x.ts).");
snippetCompletion.documentation = docs;
docs.baseUri = vscode.Uri.parse('http://example.com/a/b/c/');
```

1. 这个补全项使用到了 `commitCharacters` 属性，当按下 `.` 后将直接完成补全：

```tsx
const commitCharacterCompletion = new vscode.CompletionItem('console');
commitCharacterCompletion.commitCharacters = ['.'];
commitCharacterCompletion.documentation = new vscode.MarkdownString('Press `.` to get `console.`');
```

1. 这个补全项使用 `kind` 指定了它的类型，同时因为设置了 `command` 属性，当完成补全后将触发 `editor.action.triggerSuggest` ,也就是会再次触发补全提示：

```tsx
const commandCompletion = new vscode.CompletionItem('new');
commandCompletion.kind = vscode.CompletionItemKind.Keyword;
commandCompletion.insertText = 'new ';
commandCompletion.command = { command: 'editor.action.triggerSuggest', title: 'Re-trigger completions...' };
```

### Sample Provider2：

`provideCompletionItems()` 方法中的 `position` 参数用来表示光标的位置，`document.lineAt()` 可以获取到这一行从起始位置到光标位置的文档对象，从而提取到这一行的文本内容，在使用正则、前缀、后缀等方式来验证和分析内容：

```tsx
const linePrefix = document.lineAt(position).text.slice(0, position.character);
if (!linePrefix.endsWith('console.')) {
	return undefined;
}
```

如果文本的结尾符合 `console.` 那么将提供 `log`、`warn` 和 `error` 三个补全项供选择：

```tsx
new vscode.CompletionItem('log', vscode.CompletionItemKind.Method),
new vscode.CompletionItem('warn', vscode.CompletionItemKind.Method),
new vscode.CompletionItem('error', vscode.CompletionItemKind.Method),
```