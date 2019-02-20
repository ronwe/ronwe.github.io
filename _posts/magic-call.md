---
title: 魔术方法 
---
### FF的__noSuchMethod__没普及开，要实现magic_call的效果可以使用Proxy达到目的。###

```javascript
class MagicClass {
    constructor() {
        return new Proxy(this, {
            get(target, name) {
                if (name in target) {
                    return target[name]
                }
                return (...args) => {
                    args.unshift(name)
                    target.__noSuchMethod__.apply(target, args)
                } 
            }
        });
    }
    __noSuchMethod__(name, ...args) {
        console.log(name, args)
    }
}

let inst = new MagicClass
inst.hello('a')
```

