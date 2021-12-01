# highlightjs-lineToDiv
Plugin for highlightJS that transforms a line into a div element.

# Motivation
Like many others plugins, this is to allow add line numbers to code. If every line is at one div, than we can use css `::before` to automatically add the numbers. Additionally, if there is an ID for each line, then it is possible to highlight an entire line with some javascript code. It can be usefull to highlight errors locations for user, as used in SmartC (https://github.com/deleterium/SmartC).

# Usage
Copy the code below, paste in your file and configure values. This plugin will remove the newlines, so ensure that the div class is set to `display: block;` in css.

# Configuration
Define values for className or idName if you want these properties to be present in output div for each line. For idName, use `%line%` and it will be replaced to current line number. Do not comment all line, jKeep variable declarations if not using them (or set undefined).

```javascript
hljs.addPlugin({
    'after:highlight': (result) => {
        let className, idName
        /* Configuration */
        className = 'line'
        // idName = 'code%line%'
        /* end of configuration */
        const htmlLines = result.value.split('\n')
        let spanStack = []
        result.value = htmlLines.map((content, index) => {
            let startSpanIndex, endSpanIndex
            let needle = 0
            content = spanStack.join('') + content
            spanStack = []
            do {
                const remainingContent = content.slice(needle)
                startSpanIndex = remainingContent.indexOf('<span')
                endSpanIndex = remainingContent.indexOf('</span')
                if (startSpanIndex === -1 && endSpanIndex === -1) {
                    break
                }
                if (endSpanIndex === -1 || (startSpanIndex !== -1 && startSpanIndex < endSpanIndex)) {
                    const nextSpan = /<span .+?>/.exec(remainingContent)
                    if (nextSpan === null) {
                        // never: but ensure no exception is raised if it happens some day.
                        break
                    }
                    spanStack.push(nextSpan[0])
                    needle += startSpanIndex + nextSpan[0].length
                } else {
                    spanStack.pop()
                    needle += endSpanIndex + 1
                }
            } while (true)
            if (spanStack.length > 0) {
                content += Array(spanStack.length).fill('</span>').join('')
            }
            let retString = '<div '
            if (idName !== undefined) {
                retString += 'id="' + idName.replace('%line%', index + 1) + '" '
            }
            if (className !== undefined) {
                retString += `class="${className}"`
            }
            retString += `>${content}</div>`
            return retString
        }).join('')
    }
})
```
