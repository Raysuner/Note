用于默认导出函数的声明，会将文件夹名称当成组件名称
```json
"Export default tsx": {
  "prefix": "edf",
  "body": [
    "export default function ${1:${TM_DIRECTORY/.*\\/([^\\/]+)$/${1:/capitalize}/}} (${2:props}) {",
    "  $3",
    "}"
  ],
  "description": "Export default tsx"
}
```

用于默认导出函数的声明
```json
	"Export default function": {
		"prefix": "edf",
		"body": [
			"export default function ${1:${TM_DIRECTORY/.*\\/([^\\/]+)$/$1/}} (${2:props}) {",
			"  $3",
			"}"
		],
		"description": "Export default function"
	}
```