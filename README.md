# tiny-http-server

A small HTTP server built from scratch on top of raw TCP sockets, using Node's built-in `node:net` module. No Express and no `node:http`: every request is parsed by hand and every response is written byte by byte.

I built this to understand what frameworks like Express do under the hood.

## Running it

Requires Node.js 22.18 or newer, which can run TypeScript files directly.

```bash
npm install
node --watch server.ts
```

Then open http://localhost:3000.

## What it does

- Listens for TCP connections on port 3000
- Reads the raw HTTP request text and pulls out the method and path
- Serves files from the `public` folder:
  - `/` → `public/index.html`
  - `/about` → `public/about.html` (clean URLs, `.html` is added automatically)
  - `/style.css` → `public/style.css`
- Sets `Content-Type` from the file extension (HTML, CSS, JPEG, PNG, GIF, SVG, and more)
- Sends files as raw bytes, so images and other binary files arrive intact
- Returns `404 Not Found` for missing files
- Returns `403 Forbidden` for any path that tries to escape the `public` folder

## Project structure

```
server.ts      the whole server
public/        the files it serves
```

## What I learned

- **HTTP is just text.** A request is a few lines of plain text over a TCP connection, and a response is a status line, some headers, a blank line, then the body. Every line ends with `\r\n`.
- **Characters are not bytes.** `Content-Length` counts bytes. An emoji like 👋 is 2 characters in JavaScript but 4 bytes in UTF-8, so using `string.length` cuts off the end of the response.
- **Encodings matter.** Without `charset=utf-8` in the `Content-Type`, the browser guesses the wrong encoding and emoji turn into garbled characters (mojibake).
- **Not every file is text.** Reading an image as UTF-8 replaces invalid bytes with `�` and corrupts it. Files have to be read and sent as raw bytes (Buffers).
- **A file's extension is just a label.** The bytes decide what a file really is. For example, every PNG starts with `89 50 4E 47`, and every JPEG with `FF D8`.
- **Path traversal.** A request like `/../server.ts` could read files outside the `public` folder, including the server's own source code. The fix is to resolve the full path first, then refuse anything that doesn't stay inside `public`.
- **Browsers cache aggressively,** especially favicons. When something doesn't update, try a private window or a hard refresh before blaming the code.

## Image credits

- `sky.jpg`: photo by Alexander Lunyov on Unsplash
- `parakeet.gif`: by kuzu420 on Pixabay
