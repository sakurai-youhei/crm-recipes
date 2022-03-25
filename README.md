# crm-recipes

## markdown
```js
const context = {
    // 'reset': false, 'offset': 0,  'insertBefore': '[^',  'insertAfter': ']';
    // 'reset': true,  'offset': -1, 'insertBefore': '[',   'insertAfter': ']()';
    // 'reset': true,  'offset': 1,  'insertBefore': '[](', 'insertAfter': ')';
    // 'reset': false, 'offset': 0,  'insertBefore': '**',  'insertAfter': '**';
    // 'reset': false, 'offset': 0,  'insertBefore': '`',   'insertAfter': '`';
    // 'reset': true,  'offset': 0,  'prependEveryLine': '> ';
};

const script = document.createElement('script');
script.type = 'text/javascript';
script.src = 'https://sakurai-youhei.github.io/crm-recipes/markdown.js?t=' + Date.now();
script.onload = () => {
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
};
document.head.appendChild(script);
```
