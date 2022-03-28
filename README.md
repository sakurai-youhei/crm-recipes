# crm-recipes

## markdown
```js
const context = {
    // 'reset': false, 'offset': 0,  'insertBefore': '[^',  'insertAfter': ']'
    // 'reset': true,  'offset': -1, 'insertBefore': '[',   'insertAfter': ']()'
    // 'reset': true,  'offset': 1,  'insertBefore': '[](', 'insertAfter': ')'
    // 'reset': false, 'offset': 0,  'insertBefore': '**',  'insertAfter': '**'
    // 'reset': false, 'offset': 0,  'insertBefore': '`',   'insertAfter': '`'
    // 'reset': true,  'offset': 0,  'prependEveryLine': '> '
};

class Formatter {
    constructor({ reset, offset, insertBefore, insertAfter, prependEveryLine } = {}) {
        this.reset = reset;
        this.offset = offset;
        this.insertBefore = insertBefore;
        this.insertAfter = insertAfter;
        this.prependEveryLine = prependEveryLine;
    }

    cursor(start, end) {
        if (!this.reset) {
            return {
                'start': start + this.insertBefore.length + this.offset,
                'end': end + this.insertBefore.length + this.offset
            };
        } else if (this.offset < 0) {
            return {
                'start': end + this.insertBefore.length + this.insertAfter.length + this.offset,
                'end': end + this.insertBefore.length + this.insertAfter.length + this.offset
            };
        } else {
            return {
                'start': start + this.offset,
                'end': start + this.offset
            };
        }
    }

    transform(text) {
        if (typeof this.prependEveryLine !== "undefined") {
            return text.split(/\n/).map(line => this.prependEveryLine + line).join('\n');
        } else {
            return this.insertBefore + text + this.insertAfter;
        }
    }
}

class TextAreaFormatter extends Formatter {
    run(element) {
        const start = element.selectionStart, end = element.selectionEnd;
        element.value = element.value.slice(0, start) + this.transform(element.value.slice(start, end)) + element.value.slice(end);
        const cur = this.cursor(start, end);
        element.selectionStart = cur.start;
        element.selectionEnd = cur.end;
    }
}

class SelectionFormatter extends Formatter {
    run(selection) {
        const range = selection.getRangeAt(0);
        const start = range.startOffset, end = range.endOffset;
        if (range.startContainer === range.endContainer) {
            range.startContainer.replaceData(start, end - start, this.transform(range.startContainer.data.slice(start, end)));
        } else {
            let start2end = false;
            Object.values(range.commonAncestorContainer.childNodes).filter((el) => {
                start2end = el == range.startContainer.parentElement ? !start2end : start2end;
                if (start2end) {
                    start2end = el == range.endContainer.parentElement ? !start2end : start2end;
                    return true;
                } else {
                    return false;
                }
            }).forEach((el) => {
                el.innerText = this.transform(el.innerText);
            });
        }
        const cur = this.cursor(start, end);
        range.setStart(range.startContainer, cur.start);
        range.setEnd(range.startContainer, cur.end);
    }
}

let selection = window.getSelection(), element = document.activeElement;
while (element.contentWindow) {
    selection = element.contentWindow.getSelection();
    element = element.contentDocument.activeElement;
}
if (element instanceof HTMLTextAreaElement) {
    new TextAreaFormatter(context).run(element);
} else if (selection.rangeCount) {
    new SelectionFormatter(context).run(selection);
}
```
