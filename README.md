Cached passthrough HTTP mirror
==============================
Very simple HTTP passthrough mirror (proxy) with an aggresive cache for large files (ignores cache headers and caches everything larger than a specified minimum size). Originally written to cache large files to accelerate some repetitive operations.

Writes to the cache and serves the large files at the same time ([stream-copy](https://github.com/alexmingoia/stream-copy)).

Uses [http-proxy](https://github.com/nodejitsu/node-http-proxy) for the HTTP proxy part.

This cache saves bandwidth not latency.

It replicates the directory structure of the target server, only for the cached files (which meet or exceed `nBytesMinimumFileSize`).

The `Content-length` and `Last-modified` headers (from a `HEAD` request to the target server) are used to determine if the cached file is to be invalided. __The `HEAD` request is always made__ and so is depended upon (to proxy updated headers and for immediate cache invalidation).

There are no plans to add non-ASCII characters support or saving of headers, or 100% independent mirror capabilities.

So far tested on Windows and Ubuntu. Should work without issues on all platforms though.

__Security WARNING:__ HTTP authorization skip: when the HEAD request fails either with a non-200 HTTP status code or at network level (or something else), and the file is served directly from the cache storage, there will be no HTTP authorization. This might be fixed in the future, but at the present time, it presents a risk where security matters.

@TODO: write tests, examples, CLI endpoint, etc.

Usage
=====
```JavaScript
	const HTTPProxyCache = require("http-proxy-cache-lf");
	const http = require("http");

	const httpServer = http.createServer();

	const httpProxyCache = new HTTPProxyCache(
		/*strTargetURLBasePath*/ "http://kakao.go.ro/", 
		/*nBytesMinimumFileSize*/ 4 * 1024 * 1024 /*4 MB*/, 
		/*strCacheDirectoryRootPath*/ "/tmp/repo-cache"
	);

	httpServer.on(
		"request",
		async (incomingMessage, serverResponse) => {
			await httpProxyCache.processHTTPRequest(incomingMessage, serverResponse);
		}
	);

	httpServer.listen(8008, "127.0.0.1");
```
