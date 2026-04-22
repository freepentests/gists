# Deobfuscating JavaScript code obfuscated by obfuscator.io

Obfuscator.io is probably the most insecure JavaScript obfuscation tool I've ever seen in my life. It doesn't even do the bare minimum; it performs very basic and reversible encryption, it performs little to no string encoding, and it doesn't even randomize the original variable names.

Even though obfuscator.io has been proven to suck, it is still one of the most used obfuscators that exists. Therefore, I have written this article to explain how it can easily be reversed.

> Obfuscator.io now has a more secure obfuscation method called VM Obfuscation, but the free plan has strict limits, and nobody is going to pay money to obfuscate JavaScript code, so everyone will just end up using standard obfuscation anyway.

I know there are already sites out there such as deobfuscate.io that are designed for deobfuscating code like this, but I want to explain how it can be reversed manually.

## A step-by-step walkthrough

I am going to walk through the deobfuscation process step-by-step. I will start by creating a simple script, then I will obfuscate it with obfuscator.io, then I will deobfuscate it to get back the original code.

For simplicity, I am just going to obfuscate this code:

```js
const x = 11;

if (x > 10) console.log('hello world');
```

When passed through obfuscator.io, that transforms into this:

```js
const _0x13edb=_0x2e7e;(function(_0xead317,_0x92a624){const _0x95be55=_0x2e7e,_0x5c62e4=_0xead317();while(!![]){try{const _0x1633bd=-parseInt(_0x95be55(0x113))/0x1+-parseInt(_0x95be55(0x110))/0x2+parseInt(_0x95be55(0x10d))/0x3*(-parseInt(_0x95be55(0x112))/0x4)+parseInt(_0x95be55(0x111))/0x5*(-parseInt(_0x95be55(0x119))/0x6)+parseInt(_0x95be55(0x10e))/0x7*(-parseInt(_0x95be55(0x10f))/0x8)+parseInt(_0x95be55(0x116))/0x9*(parseInt(_0x95be55(0x118))/0xa)+-parseInt(_0x95be55(0x114))/0xb*(-parseInt(_0x95be55(0x117))/0xc);if(_0x1633bd===_0x92a624)break;else _0x5c62e4['push'](_0x5c62e4['shift']());}catch(_0x2f3949){_0x5c62e4['push'](_0x5c62e4['shift']());}}}(_0x5279,0xc9389));function _0x5279(){const _0x30158c=['228924dsoSiN','5220645fxQRuJ','2697872bYwiaw','1393223zqfIbv','15917iMdYAs','log','5651388gIOJEB','29508mquqFH','20AVDBry','6AizStw','hello\x20world','3HWQwrW','7FFeuzX','6108432WqtrEF'];_0x5279=function(){return _0x30158c;};return _0x5279();}const x=0xb;function _0x2e7e(_0x4d5564,_0x4f30d0){_0x4d5564=_0x4d5564-0x10c;const _0x52797a=_0x5279();let _0x2e7e17=_0x52797a[_0x4d5564];return _0x2e7e17;}if(x>0xa)console[_0x13edb(0x115)](_0x13edb(0x10c));
```

> I suggest you follow along with the obfuscated code above

---

## Skimming Through The Code

The code might look daunting at first, but it's actually really simple. When unminified, we can notice that there is an immediately-invoked function expression at the top of the file, accepting two arguments - `_0xead317` and `_0x92a624`.

```js
const _0x13edb = _0x2e7e;
(function (_0xead317, _0x92a624) {
    const _0x95be55 = _0x2e7e,
        _0x5c62e4 = _0xead317();
    while (!![]) {
        try {
            const _0x1633bd =
                -parseInt(_0x95be55(0x113)) / 0x1 +
                -parseInt(_0x95be55(0x110)) / 0x2 +
                (parseInt(_0x95be55(0x10d)) / 0x3) * (-parseInt(_0x95be55(0x112)) / 0x4) +
                (parseInt(_0x95be55(0x111)) / 0x5) * (-parseInt(_0x95be55(0x119)) / 0x6) +
                (parseInt(_0x95be55(0x10e)) / 0x7) * (-parseInt(_0x95be55(0x10f)) / 0x8) +
                (parseInt(_0x95be55(0x116)) / 0x9) * (parseInt(_0x95be55(0x118)) / 0xa) +
                (-parseInt(_0x95be55(0x114)) / 0xb) * (-parseInt(_0x95be55(0x117)) / 0xc);
            if (_0x1633bd === _0x92a624) break;
            else _0x5c62e4["push"](_0x5c62e4["shift"]());
        } catch (_0x2f3949) {
            _0x5c62e4["push"](_0x5c62e4["shift"]());
        }
    }
})(_0x5279, 0xc9389);
```

A function named `_0x5279` is being passed as the first argument, and a hexadecimal number `0xc9389` (which is `824201` in decimal form) is being passed as the second argument.

Let's first take a look at `_0x5279` and see what it's doing:

```js
function _0x5279() {
    const _0x30158c = [
        "228924dsoSiN",
        "5220645fxQRuJ",
        "2697872bYwiaw",
        "1393223zqfIbv",
        "15917iMdYAs",
        "log",
        "5651388gIOJEB",
        "29508mquqFH",
        "20AVDBry",
        "6AizStw",
        "hello\x20world",
        "3HWQwrW",
        "7FFeuzX",
        "6108432WqtrEF",
    ];
    _0x5279 = function () {
        return _0x30158c;
    };
    return _0x5279();
}
```

This function just returns an array of strings, so we can rename this function to `getStrings` to make it more readable, and replace all instances of `_0x5279` with `getStrings`.

---

## Hexadecimal Numbers

By reading through the code, we can notice that there are a lot of numbers starting with '0x', which is the literal prefix for hexadecimal numbers. When we paste these hexadecimal numbers into our browser console, we get the decimal form of the number, which is easier for humans to understand.

Before moving on, you should replace all the hexadecimal numbers in the code with their decimal form. (e.g. replace 0xa with 10, replace 0x10 with 16, blah blah blah).

> Personally, if I was writing an obfuscator, I'd make all the numbers start with `0b` for binary instead.

---

## Renaming Variables

```js
const _0x13edb = _0x2e7e;
(function (_0xead317, _0x92a624) {
    const _0x95be55 = _0x2e7e,
        _0x5c62e4 = _0xead317();
    while (!![]) {
        try {
            const _0x1633bd =
			-parseInt(_0x95be55(275)) / 1 +
			-parseInt(_0x95be55(272)) / 2 +
			(parseInt(_0x95be55(269)) / 3) * (-parseInt(_0x95be55(274)) / 4) +
			(parseInt(_0x95be55(273)) / 5) * (-parseInt(_0x95be55(281)) / 6) +
			(parseInt(_0x95be55(270)) / 7) * (-parseInt(_0x95be55(271)) / 8) +
			(parseInt(_0x95be55(278)) / 9) * (parseInt(_0x95be55(280)) / 10) +
			(-parseInt(_0x95be55(276)) / 11) * (-parseInt(_0x95be55(279)) / 12);
            if (_0x1633bd === _0x92a624) break;
            else _0x5c62e4["push"](_0x5c62e4["shift"]());
        } catch (_0x2f3949) {
            _0x5c62e4["push"](_0x5c62e4["shift"]());
        }
    }
})(getStrings, 824201);
```

As you can see, there is a constant named `_0x13edb`, which serves as an alias for `_0x2e7e`. We can get rid of this line and replace all occurances of `_0x13edb` with `_0x2e7e` directly.

At the top of the IIFE, there is a function named `_0x95be55`, which also serves as an alias for `_0x2e7e`, so we're just going to repeat the process again - get rid of `_0x95be55` and replace all its occurances with `_0x2e7e`.

At the top of the IIFE, there is also another variable named `_0x5c62e4`, which is being set to the return value of `_0xead317` (which is the first argument we passed into the function, `getStrings`). We can get rid of the first argument, rename `_0x5c62e4` to `strings` and call `getStrings` directly.

After performing those 3 steps, we are now left with this code:

```js
(function (_0x92a624) {
    const strings = getStrings();
    while (!![]) {
        try {
            const _0x1633bd =
			-parseInt(_0x2e7e(275)) / 1 +
			-parseInt(_0x2e7e(272)) / 2 +
			(parseInt(_0x2e7e(269)) / 3) * (-parseInt(_0x2e7e(274)) / 4) +
			(parseInt(_0x2e7e(273)) / 5) * (-parseInt(_0x2e7e(281)) / 6) +
			(parseInt(_0x2e7e(270)) / 7) * (-parseInt(_0x2e7e(271)) / 8) +
			(parseInt(_0x2e7e(278)) / 9) * (parseInt(_0x2e7e(280)) / 10) +
			(-parseInt(_0x2e7e(276)) / 11) * (-parseInt(_0x2e7e(279)) / 12);
            if (_0x1633bd === _0x92a624) break;
            else strings["push"](strings["shift"]());
        } catch (_0x2f3949) {
            strings["push"](strings["shift"]());
        }
    }
})(824201);
```

`_0x1633bd` seems to hold the result of a complex calculation, so we're going to rename that variable `calculation`.

The function is also calling `push` and `shift` on the `strings` array, but it is using bracket notation instead of dot notation to access these methods, so we can simply just replace `["push"]` and `["shift"]` with `.push` and `.shift` instead.

## \_0x2e7e - the get string function

In the IIFE, there are a lot of calls being made to the `_0x2e7e` function. Let's inspect what this function does:

```js
function _0x2e7e(_0x4d5564, _0x4f30d0) {
    _0x4d5564 = _0x4d5564 - 268;
    const getStrings7a = getStrings();
    let _0x2e7e17 = getStrings7a[_0x4d5564];
    return _0x2e7e17;
}
```
> one of the variables is named getStrings7a because of the replace operation I did earlier; just ignore that

The second argument, `_0x4f30d0`, doesn't seem to be used at all in the code, so we can just get rid of that. The first argument, `_0x4d5564`, seems the be the only one that's actually used.

The function is reassigning `_0x4d5564` to itself minus 268, calling `getStrings`, then returning a string at a specific index of the array that `getStrings` returns.

We can rewrite this entire function like this:

```js
function getString(index) {
    const offset = 268;
    index = index - offset;

    const strings = getStrings();
    let string = strings[index];

    return string;
}
```

Now, replace all occurances of `_0x2e7e` with `getString`.

## What does the IIFE do?

We've now renamed all the meaningless variables, but it's still not quite obvious what this calculation is doing:

```js
(function () {
    const strings = getStrings();
    while (!![]) {
        try {
            const calculation =
			-parseInt(getString(275)) / 1 +
			-parseInt(getString(272)) / 2 +
			(parseInt(getString(269)) / 3) * (-parseInt(getString(274)) / 4) +
			(parseInt(getString(273)) / 5) * (-parseInt(getString(281)) / 6) +
			(parseInt(getString(270)) / 7) * (-parseInt(getString(271)) / 8) +
			(parseInt(getString(278)) / 9) * (parseInt(getString(280)) / 10) +
			(-parseInt(getString(276)) / 11) * (-parseInt(getString(279)) / 12);
            if (calculation === 824201) break;
            else strings.push(strings.shift());
        } catch {
            strings.push(strings.shift());
        }
    }
})();
```

The function seems to be modifying `strings` based on the result of the calculation, so I'm just going to put `console.log(strings)` before the conditional check, and then I'm going to run the code to analyze its behaviour:

```js
(function () {
    const strings = getStrings();
    while (!![]) {
        try {
            const calculation =
			-parseInt(getString(275)) / 1 +
			-parseInt(getString(272)) / 2 +
			(parseInt(getString(269)) / 3) * (-parseInt(getString(274)) / 4) +
			(parseInt(getString(273)) / 5) * (-parseInt(getString(281)) / 6) +
			(parseInt(getString(270)) / 7) * (-parseInt(getString(271)) / 8) +
			(parseInt(getString(278)) / 9) * (parseInt(getString(280)) / 10) +
			(-parseInt(getString(276)) / 11) * (-parseInt(getString(279)) / 12);
			
			console.log(strings); 
		
            if (calculation === 824201) break;
            else strings.push(strings.shift());
        } catch {
            strings.push(strings.shift());
        }
    }
})();
```
<img src="https://i.ibb.co/Fvp18xF/image.png">

Upon analyzing the output, we can observe that this function is performing a left-shift on the `strings` array until it is ordered correctly.

We can now get rid of all the IIFE bullshit at the top of the code, and now we are left with this:

```js
function getStrings() {
    const strings = [
  'hello world',   '3HWQwrW',
  '7FFeuzX',       '6108432WqtrEF',
  '228924dsoSiN',  '5220645fxQRuJ',
  '2697872bYwiaw', '1393223zqfIbv',
  '15917iMdYAs',   'log',
  '5651388gIOJEB', '29508mquqFH',
  '20AVDBry',      '6AizStw'
]

    getStrings = function () {
        return strings;
    };

    return getStrings();
}

const x = 11;

function getString(index) {
    index = index - 268;

    const strings = getStrings();
    let string = strings[index];

    return string;
}

if (x > 10) console[getString(277)](getString(268));

```

Much simpler, right?

## Getting our original code back

Now, we can just replace `getString(277)` and `getString(268)` with their corresponding values:
```js
if (x > 10) console['log']('hello world');
```

and then use dot notation for `console.log` instead of bracket notation:
```js
if (x > 10) console.log('hello world');
```

Now delete the `getString` and `getStrings` functions, and BOOM! We have the original code we started with:

```js
const x = 11;

if (x > 10) console.log('hello world');
```

# Why you shouldn't use obfuscator.io

The process I just showed you is not complicated. In fact, it's so simple that (https://obf-io.deobfuscate.io/)[there are even sites dedicated to automating the process of deobfuscating this shit]. What's the point of obfuscating your code when someone can just go to deobfuscate.io and get back the original code instantly? It does absolutely nothing but increase the size of your JavaScript file.

# Alternatives

In reality, I don't think you should obfuscate your code at all. Obfuscation is a form of security through obscurity, which is a widely discouraged security practice that relies on the idea that if attackers don't understand how your program works, they will have a harder time hacking it.

Although relying on security through obscurity alone is a bad practice, it can be beneficial when used in union with other security practices, so I would recommend just using js-confuser or enigma-vm if you plan on obfuscating your code.

# Conclusion

Don't use obfuscator.io.
