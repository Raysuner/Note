## stringify

`stringify`方法是把对象转换成`json`

```js
   let myJSON = {
    stringify: function (object) {
        if (typeof object === "number") {
            return String(object)
        } else if (typeof object === "string") {
            return '"' + object + '"'
        } else if (typeof object === "boolean") {
            if (object) {
                return "true"
            }
            return "false"
        } else if (Array.isArray(object)) {
            let result = "["
            for (let i = 0; i < object.length; i++) {
                let val = this.stringify(object[i])
                result +=
                    (val === "undefined" || val === "NaN" ? "null" : val) + ","
            }
            return result.slice(0, -1) + "]"
        } else if (
            Object.prototype.toString.call(object) === "[object Object]"
        ) {
            let result = "{"
            for (let key in object) {
                let val = this.stringify(object[key])
                result +=
                    '"' +
                    key +
                    '":' +
                    (val === "undefined" || val === "NaN" ? "null" : val) +
                    ","
            }
            return result.slice(0, -1) + "}"
        }
    },

    parse: function (arg) {
        function parseJson() {
            function parseTrue() {
                i += 4
                return true
            }

            function parseFalse() {
                i += 5
                return false
            }

            function parseString() {
                let ret = ""
                i++
                while (str[i] != '"') {
                    ret += str[i++]
                }
                i++
                return ret
            }

            function parseNumber() {
                let ret = ""
                while (str[i] >= "0" && str[i] <= "9") {
                    ret += str[i++]
                }
                return Number(ret)
            }

            function parseNull() {
                i += 4
                return "null"
            }

            function parseArray() {
                let ret = []
                i++
                while (str[i] !== "]") {
                    ret.push(parseJson())
                    if (str[i] === ",") {
                        i++
                    }
                }
                i++
                return ret
            }

            function parseObject() {
                let ret = {}
                i++
                while (str[i] !== "}") {
                    let key = parseJson()
                    i++
                    let val = parseJson()
                    ret[key] = val
                    if (str[i] === ",") {
                        i++
                    }
                }
                i++
                return ret
            }

            while (i < str.length) {
                if (str[i] === "{") {
                    return parseObject()
                } else if (str[i] === "[") {
                    return parseArray()
                } else if (str[i] === '"') {
                    return parseString()
                } else if (str[i] === "t") {
                    return parseTrue()
                } else if (str[i] === "f") {
                    return parseFalse()
                } else if (str[i] === "n") {
                    return parseNull()
                } else if (str[i] >= "0" && str[i] <= "9") {
                    return parseNumber()
                }
            }
        }
        let i = 0
        let str = arg
        return parseJson()
    },
}
```

上述代码没有对函数和`symbol`做处理，以后补全