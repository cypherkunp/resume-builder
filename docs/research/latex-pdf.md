# LaTeX to PDF

## Question

How can a TanStack Start app turn one LaTeX Template plus plain-text Resume Sections into a PDF whose text a parser can select and extract?

Settled constraints: there is one Template, the person never edits LaTeX, the Resume is stored only in the browser, and the PDF must stay simple enough for an applicant tracking system. This note reports which compilation approaches exist, what each requires (browser WASM, a server binary, or an external service), and whether the PDF they emit contains real extractable text rather than outlined glyphs or images of text. It does not choose where compilation should run.

TanStack Start does not document a LaTeX compiler, a WASM helper, or a PDF API. The approaches below are engines and services the app can call from client code, from a server function, or over HTTP. Start only decides where that call is allowed to run.

## What "extractable text" means

ISO 32000-1:2008 (the PDF 1.7 spec, Adobe's authorized public copy) separates painting glyphs from recovering characters.

Text is shown by text-showing operators inside a text object (`BT` … `ET`). The operators are `Tj` (show a string), `'` and `"` (show a string on the next line), and `TJ` (show strings with per-glyph position adjustments). A string operand is a sequence of character codes that select glyphs from a font. The spec says the grouping of glyphs into strings does not change how they are displayed, and that search and other extraction work better when those strings are long and in natural reading order (clause 9.4.3, notes 6 and 7).

Extraction is a second step (clause 9.10). A reader maps character codes to Unicode, in this order: a `/ToUnicode` CMap on the font, if present; otherwise a predefined simple encoding (MacRoman, MacExpert, WinAnsi) or an encoding whose differences use only standard Adobe glyph names; otherwise a predefined composite CMap or a standard Adobe CID collection. If none of those produce a Unicode value, the spec says there is no way to know what the code represents, and a conforming reader may choose a code of its own. An `ActualText` entry on structure or marked content can supply replacement text instead (clause 9.10.1, pointing at 14.9.4). Tagged PDF must provide at least one of these methods (clause 9.10.2). Ordinary extraction does not require a structure tree.

Outlined glyphs and pictures of text are different objects. Path operators paint shapes. Image XObjects paint rasters. Neither is a text-showing operator, so a parser that only reads text operators gets nothing from them. Ghostscript's `pdfwrite` family normally keeps fonts as fonts. Its `-dNoOutputFonts` switch is the documented way to throw fonts away and store text as vectors, or as bitmaps when the input font was a bitmap. That is a post-process, not what the TeX engines do by default.

No source consulted here defines an applicant-tracking-system PDF profile. "Simple enough for an ATS" is therefore not a format this note can pin to a spec. The spec facts an ATS-style parser needs are: text-showing operators, a Unicode mapping for every font that carries resume text, word breaks the parser can see, and content in reading order.

## Approaches

### 1. TeX Live engines, as an operating-system binary

TeX Live ships native executables and a `texmf` tree of macros, fonts, and configuration. The engines that write PDF are:

- **pdfTeX**, invoked for the usual `pdflatex` format. The TeX Live Guide says it writes PDF as well as DVI. The pdfTeX manual says its purpose is PDF directly from TeX source, with no separate driver. It embeds Type 1 and TrueType fonts (OpenType only in a limited full-embed mode). It does not natively accept UTF-8 input or Unicode fonts; the manual says LaTeX handles UTF-8 at macro level, and points Unicode documents at LuaTeX or XeTeX.
- **XeTeX**. CTAN describes it as a Unicode engine with OpenType, TrueType, and AAT fonts. Its own output is an extended DVI (XDV). A bound `xdvipdfmx` driver turns that into PDF. TeX Live includes both.
- **LuaTeX**. The TeX Live Guide and luatex.org describe it as an extended pdfTeX with a Lua interpreter, Unicode input, and OpenType/TrueType fonts. `lualatex` is the LaTeX format on that engine.

The same engines are what MacTeX and MiKTeX ship. The TeX Live Guide says MacTeX is TeX Live plus a Mac installer, and that the main trees and binaries are the same. CTAN's pdfTeX entry says TeX Live, MacTeX, and MiKTeX all include pdfTeX and pdfLaTeX.

**Requires:** an OS process that can execute the binary, a readable TeX tree (or a smaller scheme of one), and fonts the Template actually loads. The 2026 installer's example of the full scheme is about 9402 MB; the same screen offers medium and small schemes, whose sizes this note did not copy. The process needs a writable working directory for the filled `.tex` file and the PDF. TeX Live's security section says the engine can write files while processing a document, and that untrusted input should be compiled in a fresh directory. Shell escape is part of that risk; Tectonic's compile page (below) names it explicitly for the XeTeX-derived engine.

This is not a browser build. A TanStack Start server function can call it only on a host that can spawn that process and see the TeX tree. Start's execution model describes the server environment as a Node.js runtime with a file system, and `createServerFn` as an RPC the client invokes over the network. The hosting guide also lists Cloudflare Workers, Netlify, Vercel, and other targets, and the server entry point is a WinterCG `fetch` handler so the same app can run on edge runtimes. Those targets are not documented as Node processes with a TeX Live install. Whether a given host can run the binary is a property of the host, not of Start.

**Text in the PDF:** real text, not a page image and not outlined paths. The pdfTeX manual's minimal example is a content stream whose text sits between `BT` and `ET` as PDF strings, with a font resource and widths. Images are a separate inclusion path (`\pdfximage` and image XObjects). METAFONT bitmaps can be embedded as Type 3 fonts; the manual says they still are fonts, and strongly advises against them because they render poorly. They are not a raster of the page.

Unicode extraction is not automatic for pdfTeX. The manual says that, by default, pdfTeX does not write a `/ToUnicode` CMap. Basic ASCII search usually works; ligatures such as "fi" and anything beyond ASCII often do not, because the reader is left to guess. Setting `\pdfgentounicode=1` writes the CMap from glyph-name mappings. Those mappings come from `\pdfglyphtounicode`, and the distributed `glyphtounicode.tex` file supplies thousands of them (`\input glyphtounicode` plus `\pdfgentounicode=1`). The `cmap` package, which the manual points LaTeX users at for the same effect, preloads CMap files for OT1, T1, T2A/B/C, T5, OT6, LGR, LAE, and LFE. Its README says this is what makes pdfLaTeX output searchable and copyable. It works for Type 1 and, with recent pdfTeX, Type 3 including PK fonts. It does not work for virtual fonts; that is a pdfTeX limitation the package states, not a viewer quirk. The TeX FAQ states the same split: pdfLaTeX character codes follow the font encoding, so OT1/T1 is mostly fine for ASCII and unreliable past that, while XeTeX and LuaTeX PDFs are in good shape because those engines work in Unicode throughout. `cmap` and `mmap` are pdfTeX-only, and the FAQ says they do not cover fonts that reach the engine only through virtual fonts.

Word spaces are a further pdfTeX gap. The manual says pdfTeX output often fragments words with kerning adjustments, and that viewers ignore those adjustments when searching. It also says the engine does not, by default, put a real space character in the content stream. `\pdfinterwordspaceon` makes it guess interword gaps and insert a space character (from the current font if the encoding has a `/space` at slot 32, otherwise from a fallback `pdftexspace` font). The manual says this improves extraction and is required by PDF/A. It does not change how the page looks.

XeTeX and LuaTeX still emit text operators and embedded fonts; they are not outline or bitmap exporters. The TeX FAQ's Unicode claim is the strongest primary statement found that their copy-paste text is reliable. This note did not find a sentence in the XeTeX or LuaTeX manuals that guarantees a complete `/ToUnicode` CMap for every font. The XeTeX mailing list has reports of `xdvipdfmx` failing to load a ToUnicode CMap for particular OpenType fonts, so a Unicode engine is not a proof that every glyph maps. For a Template that uses one Unicode OpenType text font and ordinary Latin text, the FAQ's claim is the relevant one. A Template that draws text with TikZ paths, `\pdfliteral`, or included images would leave that text out of the text operators; that would be the Template, not the engine.

### 2. Tectonic, as a native binary or embeddable library

Tectonic is a separate program built from XeTeX and TeX Live. Its site says it converts TeX files to PDF, downloads support files on demand from a bundle (so a full TeX Live install is not required), loops TeX and BibTeX itself, and uses XeTeX's Unicode and OpenType behavior. The final step in its own sample transcript is `xdvipdfmx` writing the PDF. One executable contains the engine. The bundle is cached after the first run; a compile can also be pointed at a local bundle (`tectonic -X compile --bundle …`). The engine is also published as a Rust library so another program can embed it. Current install documentation ships prebuilt native executables (and package-manager or `cargo` builds). The contribute page says a fuller Rust port would eventually allow WebAssembly, and that this has not happened. No released WASM build is documented. A browser build of Tectonic is unverified.

**Requires:** the `tectonic` executable (or the Rust crate linked into a server process), a network fetch of the bundle on first use unless the bundle is vendored, and a writable output directory. Default compile mode is "trusted", which the compile reference says can enable engine features that are unsafe on untrusted input; shell escape is the example they give. User-written Section text is untrusted TeX input even though the person never edits the Template.

**Text in the PDF:** the PDF is the XeTeX/`xdvipdfmx` PDF described above. Tectonic does not document a mode that outlines glyphs or rasterizes the page. The same Unicode-mapping caveats as XeTeX apply. This note did not find a Tectonic statement that adds or removes `/ToUnicode`.

### 3. WebAssembly ports of the same engines

These run the engines above inside a WASM runtime. The PDF is the engine's PDF, not a screenshot, to the extent the port is faithful. None of them is a TanStack API. The browser can load them from client-only code. A server-side WASM runtime would be a different host, and the projects below do not all document one.

**SwiftLaTeX** compiles pdfTeX and XeTeX to WebAssembly and runs them in the browser. The project says all computation is local, and that the PDF matches TeX Live or MiKTeX. The library is a script plus WASM: `loadEngine`, `writeMemFSFile` for the Template and any assets, `setEngineMainFile`, `compileLaTeX`. The result is a PDF `Uint8Array` and a log. Packages are not inside the WASM module. The engine fetches them from CTAN or from `https://texlive.swiftlatex.com`, or from a server you host with their Texlive-Ondemand project (`setTexliveEndpoint`). They recommend the XeTeX build for UTF-8 and OpenType, and note that it does not ship a full ICU dataset, so locale line breaking can differ. pdfTeX is the build they suggest for English. The repository license is AGPL-3.0. Browser use is the documented target. An open issue asks about Node.js; the project has not documented Node support, so server-side SwiftLaTeX is unverified.

**texlive.js** is an Emscripten port of TeX Live 2016 pdfTeX. Its README says it creates PDFs from LaTeX in the browser and supports packages. The demo constructs a `PDFTeX` object and compiles a source string. Package files live in a virtual filesystem the wiki explains how to extend. It is GPL-2.0 and frozen at TeX Live 2016. It is a real compiler, but it is nine years behind current TeX Live.

**busytex** compiles TeX Live 2023 programs (`pdftex`, `xetex`, `luahbtex`, `xdvipdfmx`, `bibtex8`, `makeindex`, and kpathsea tools) to a static x86_64-linux binary and to WASM32. The README's WASM path downloads the module plus separate TeX Live data packages (`texlive-basic`, latex-recommended, latex-extra, science) and serves them to a page. The data files are large and are not optional if the Template needs those packages. The repo's own code is MIT; the binaries include TeX Live code under TeX Live's licenses.

**Text in the PDF:** SwiftLaTeX's claim of the same output as TeX Live or MiKTeX means the text-operator and `/ToUnicode` behavior of the engine you select, including pdfTeX's default omission of `/ToUnicode`. texlive.js and busytex are ports of pdfTeX, XeTeX, and (busytex only) LuaTeX, so the same engine rules apply. This note did not find a statement from any of the three that they convert text to outlines or to page images. SwiftLaTeX still needs a network (or a self-hosted file server) for TeX files it does not already have in memory, even though the compile itself is local. That fetch is packages and fonts, not the Resume, unless the Template is left on that server.

### 4. An external HTTP compiler

The app sends the filled Template to someone else's TeX Live and gets a PDF back. The Resume text leaves the browser. That is required by the service, not by Start.

**LaTeX-on-HTTP** (`latex-on-http`, live beta at `https://latex.ytotech.com`) is an HTTP API in front of TeX Live. `POST /builds/sync` takes JSON resources and a `compiler` of `pdflatex` (the default), `xelatex`, `lualatex`, `platex`, `uplatex`, or `context`, and returns the PDF. The project's own description is TeX Live on a server plus an HTTP API. The README says the API is experimental and likely to change. A TUGBoat paper by the author states the same architecture and names Overleaf's CLSI and LaTeX-Online as prior art.

**LaTeX.Online** (`latex-online`, `https://latexonline.cc`) compiles TeX from a URL, from raw text (`/compile?text=`), or from a git repo. Optional `command` is `pdflatex`, `xelatex`, or `lualatex`. Its README says success is the PDF and failure is a 4xx with the log. A self-hosted Docker image exists. The hosted service's deployment dependencies include a `pdflatex` binary. The text endpoint puts the document in the query string.

**Overleaf CLSI** is a REST API (`POST /project/<id>/compile`) whose `compiler` option is `latex`, `pdflatex`, `xelatex`, or `lualatex`. The README describes it running TeX Live inside a Docker image and returning a URL for the PDF. It is software you host (or run inside Overleaf). This note did not find an Overleaf public compile API that a third-party app can call without operating that service. Treat "use Overleaf's cloud" as unverified.

**Text in the PDF:** whatever engine the request selects, with that engine's extractability rules above. The services document PDF output from TeX Live compilers. They do not document outlining or rasterizing the page.

### 5. Not a compiler for this question

**LaTeX.js** (`latexjs/latexjs`, linked from busytex's references) and similar in-browser LaTeX-to-HTML tools do not produce a PDF. They are out of scope.

**web2js / dvi2html** (also listed by busytex) turn TeX into DVI or HTML in the browser. They are not PDF compilers.

**Ghostscript** and other PDF post-processors can destroy extractable text after a good compile. `-dNoOutputFonts` stores text as vectors or bitmaps. A raster device paints pixels. Neither belongs in the pipeline if a parser must select the text.

## Implications for the later decision

These are facts that ticket needs. They are not a recommendation.

- The compile input is TeX source. The pdfTeX manual describes PDF "directly from TeX source files." Section text is plain text in the glossary sense. Splicing it into the Template makes it TeX input. Characters that TeX treats as special will not survive as literal text unless the filler or the Template escapes them. This note did not re-quote the category-code table; the decision still has to treat escaping as part of filling the Template, independent of where the engine runs.
- pdfTeX's default PDF is real, selectable text for simple ASCII and a poor Unicode extraction for ligatures and non-ASCII, until the Template (or a format built around it) turns on `/ToUnicode` via `glyphtounicode` or `cmap`, and, if a parser should see U+0020 rather than a gap, `\pdfinterwordspaceon`. XeTeX and LuaTeX are the engines the pdfTeX manual and the TeX FAQ point at for Unicode. SwiftLaTeX's own advice matches that: XeTeX for UTF-8, pdfTeX for English. The Template's font has to be one the chosen engine embeds with a Unicode mapping. Virtual fonts defeat `cmap`. Type 3 bitmap fonts are a documented pdfTeX fallback to avoid.
- ISO 32000-1 says extraction works better when text strings are long and in reading order. A single-column flow of text operators matches that. Text placed by absolute moves, or drawn as paths, does not. Tagging is a separate, stricter requirement (tagged PDF must expose a Unicode mapping). Nothing in these sources says an ATS requires tagged PDF. Nothing in these sources defines the ATS check beyond extractable characters.
- Where the bytes are produced does not change the PDF rules. It changes the runtime. A native binary (TeX Live or Tectonic) needs an OS process, a TeX tree or a Tectonic bundle, and a host that is allowed to spawn processes. Start's server functions are how client code would call that host, and calling them sends the filled source across the network. Workers-style and other WinterCG targets are fetch handlers; Start documents them as deployment targets and does not document a TeX Live process there. WASM (SwiftLaTeX, texlive.js, busytex) runs where a WASM runtime and the TeX data files are available. In the browser that can be client-only code, so the Resume need not leave the machine, but SwiftLaTeX still fetches missing package files from a TeX endpoint, and busytex's data packages have to be hosted somewhere the page can load. An external service always receives the Resume text. SwiftLaTeX is AGPL; shipping its engine is a license constraint the decision has to read.
- Untrusted Section text is still TeX. TeX Live warns that processing a document can write files. Tectonic's default trusted mode can enable shell escape. A compile of user-filled input needs the engine's untrusted or restricted mode, whichever the chosen tool actually implements. This note did not compare those flags across pdfTeX, XeTeX, LuaTeX, and Tectonic.
- Tectonic on WebAssembly, SwiftLaTeX under Node, and a public Overleaf compile API were not established by the projects' own docs. They should not be treated as available approaches.

## Sources

- TanStack Start execution model: https://tanstack.com/start/latest/docs/framework/react/guide/execution-model
- TanStack Start server functions: https://tanstack.com/start/latest/docs/framework/react/guide/server-functions
- TanStack Start server entry point: https://tanstack.com/start/latest/docs/framework/react/guide/server-entry-point
- TanStack Start hosting: https://tanstack.com/start/v0/docs/framework/react/guide/hosting
- ISO 32000-1:2008 (Adobe public copy), clauses 9.4.3 and 9.10: https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf
- pdfTeX user manual (source `pdftex.tex`, pdfTeX 1.40.29, 16 February 2026), especially the PDF introduction, font setup, `\pdfgentounicode`, `\pdfglyphtounicode`, and `\pdfinterwordspaceon`: https://mirrors.ibiblio.org/CTAN/systems/doc/pdftex/manual/pdftex.tex and https://pdftex.org
- CTAN pdfTeX package entry (distributions that include it): https://ctan.org/pkg/pdftex
- TeX Live Guide 2026 (engines, install, security, MacTeX): https://www.tug.org/texlive/doc/texlive-en/texlive-en.html
- CTAN XeTeX entry: https://ctan.org/pkg/xetex
- LuaTeX home (engine description): https://www.luatex.org/
- TeX FAQ, "Copy-paste-able/searchable PDF files": https://texfaq.org/FAQ-cpy-srchpdf
- `cmap` package README: https://mirror.apps.cam.ac.uk/pub/tex-archive/macros/latex/contrib/cmap/README and https://ctan.org/pkg/cmap
- XeTeX list note of a missing ToUnicode CMap (caveat, not the normal case): https://tug.org/pipermail/xetex/2014-July/025391.html
- Tectonic site: https://tectonic-typesetting.github.io/en-US
- Tectonic introduction (XeTeX, library, bundle): https://tectonic-typesetting.github.io/book/latest/introduction/
- Tectonic install (native executable only): https://tectonic-typesetting.github.io/book/latest/installation/
- Tectonic `compile` (PDF output, trusted mode and shell escape): https://tectonic-typesetting.github.io/book/latest/v2cli/compile.html
- Tectonic contribute page (WebAssembly named as a future target): https://tectonic-typesetting.github.io/en-US/contribute.html
- SwiftLaTeX README: https://github.com/SwiftLaTeX/SwiftLaTeX/blob/master/README.md
- SwiftLaTeX Texlive-Ondemand: https://github.com/SwiftLaTeX/Texlive-Ondemand
- SwiftLaTeX Node.js issue (open, undocumented): https://github.com/SwiftLaTeX/SwiftLaTeX/issues/78
- texlive.js README: https://github.com/manuels/texlive.js/blob/master/README.md
- busytex README: https://github.com/busytex/busytex/blob/master/README.md
- LaTeX-on-HTTP README: https://github.com/YtoTech/latex-on-http/blob/master/README.md
- TUGBoat 41:2, Yoan Tournade, "LaTeX-on-HTTP": https://www.tug.org/TUGboat/tb41-2/tb128tournade-http.pdf
- LaTeX.Online README: https://github.com/aslushnikov/latex-online/blob/master/README.md
- Overleaf CLSI README (self-hosted TeX Live HTTP API): https://github.com/overleaf/clsi/blob/master/README.md
- Ghostscript high-level devices, `-dNoOutputFonts`: https://ghostscript.com/docs/9.56.1/VectorDevices.htm
